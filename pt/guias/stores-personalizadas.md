# Escrevendo uma store personalizada

Você pode usar qualquer motor de armazenamento por trás do CacheerPHP sem tocar no
núcleo nem ler o código das stores nativas. Implemente o contrato `Store` de quatro
métodos, adicione interfaces de capacidade para o que suportar, e comprove com a suíte
de conformidade compartilhada.

## 1. A store mínima

```php
use Silviooosilva\CacheerPhp\Contracts\{Store, Clock};
use Silviooosilva\CacheerPhp\Kernel\{CacheEntry, Key, Ttl};

final class MyStore implements Store
{
    public function __construct(private readonly Clock $clock) {}

    public function get(Key $key): CacheEntry
    {
        // busque por $key->identity(); num hit vivo:
        //   return CacheEntry::hit($key, $value, $createdAt, $expiresAt); // $expiresAt null = para sempre
        return CacheEntry::miss($key);
    }

    public function set(Key $key, mixed $value, Ttl $ttl): void
    {
        $expiresAt = $ttl->expiresAt($this->clock); // null = para sempre
        // persista...
    }

    public function delete(Key $key): bool { /* true se removeu algo */ }

    public function clear(): void { /* apenas seu keyspace configurado */ }
}
```

Regras que a suíte de conformidade impõe:

- **O tempo vem do `Clock` injetado** — nunca chame `time()` direto. É isso que torna
  o comportamento testável com um clock falso.
- **A expiração é preguiçosa.** `get()` em uma entrada expirada retorna um miss.
- **Valores fazem round-trip sem perdas**, incluindo `false`, `0`, `''`, `[]` e
  estruturas aninhadas. Use o pipeline de armazenamento em vez de serialização própria.
- **`clear()` e toda varredura ficam dentro do seu keyspace** — nunca toque em dados
  não relacionados num backend compartilhado.

## 2. Adicione só as capacidades que você garante

Implemente as [interfaces de capacidade](../api/drivers.md#interfaces-de-capacidade)
que você realmente garante. O núcleo lança `UnsupportedCapabilityException` para as
demais em vez de fingir.

### Nunca responda perguntas de capacidade com `instanceof`

Um decorator precisa declarar toda capacidade que possa repassar, então `instanceof`
é `true` para wrappers em volta de stores que não a honram. Pergunte com
`Capabilities::supports($store, X::class)` (ou `$cache->supports(X::class)`).

Se a **sua** store for ela mesma um decorator, implemente `CapabilityAware` e delegue
para a store que vai executar a operação:

```php
use Silviooosilva\CacheerPhp\Contracts\CapabilityAware;
use Silviooosilva\CacheerPhp\Kernel\Capabilities;

final class MeuDecorator implements Store, AtomicStore, LockingStore, CapabilityAware
{
    public function supports(string $capability): bool
    {
        return Capabilities::supports($this->inner, $capability);
    }
}
```

O kernel depende disso para degradar otimizações opcionais em vez de falhar — sem
isso, embrulhar uma store pode transformar um `remember()` funcional em exceção.

## 3. Reutilize o pipeline de armazenamento

Codifique valores por um `EnvelopeCodec` de um
[`PipelineConfig`](../api/configuracao.md) para ganhar serialização, compressão
opcional, criptografia autenticada, limites de tamanho e compatibilidade de leitura v5
de graça:

```php
$codec = PipelineConfig::default()->withGzip()->codec();
$blob  = $codec->encode($value);   // envelope versionado
$value = $codec->decode($blob);    // falha tipada em adulteração / acima do limite
```

## 4. Comprove com a suíte de conformidade

Estenda `Tests\Support\StoreConformance` e retorne sua store de `createStore()`. Ela
roda todo o contrato base mais cada bloco de capacidade que sua store declara, e pula
os que ela não declara:

```php
use Tests\Support\{FakeClock, StoreConformance};
use Silviooosilva\CacheerPhp\Contracts\Store;

final class MyStoreConformanceTest extends StoreConformance
{
    protected function createStore(FakeClock $clock): Store
    {
        return new MyStore($clock);
    }
}
```

Se passar, sua store compõe com `Cacheer`, escopos, camadas, resiliência, os adaptadores
PSR e a CLI exatamente como as nativas.

## 5. Sendo listado como compatível

Um adaptador da comunidade é listado como compatível quando passa a suíte de
conformidade em CI nas versões de PHP suportadas, documenta quais capacidades oferece e
suas garantias, e documenta seus modos de falha (perda de conexão, timeouts). Veja o
guia `WRITING_A_STORE.md` e o template de proposta de driver no repositório do pacote.
