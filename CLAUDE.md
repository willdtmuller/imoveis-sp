# Imóveis SP — base de dados

Fonte de verdade: array `window.PROPERTIES` dentro de `imoveis.html` (Opção A, sem vínculo externo).
A planilha é só export pontual (botão Exportar → CSV → importar no Google Sheets). Não há sincronização de volta.

## Esquema de cada imóvel

```js
{
  id: "string-unico",            // ex: "qa-481923" ou slug do anúncio
  corretor_nome: "string",
  corretor_tel: "55DDDNNNNNNNNN", // só dígitos, formato wa.me (55 + DDD + número)
  bairro: "string",
  endereco: "Rua, número - Bairro, São Paulo - SP",
  lat: -23.0, lng: -46.0,        // geocodificado na ingestão
  area_m2: 0,
  quartos: 0,
  infra: ["Piscina", "Academia", ...], // lista de strings (vocabulário ainda aberto)
  custo_total: 0,                // pacote fechado: aluguel + condomínio + IPTU SOMADOS
  link: "url do anúncio",
  visita: false,
  visita_inicio: "",             // ISO local "AAAA-MM-DDTHH:MM" ou "" se sem visita
  visita_min: 45,
  proposta: "",                  // preenchido manualmente depois
  comentarios: ""
}
```

Maps e Agenda NÃO são armazenados — o HTML os gera a partir de `lat/lng`, `endereco` e `visita_inicio`.

## Protocolo de ingestão (quando eu mandar um link)

1. Buscar a página (`web_fetch` / leitura de HTML). Procurar dados estruturados (JSON-LD) primeiro.
2. Extrair: bairro, endereço, área, quartos, infra, e os valores de aluguel/condomínio/IPTU.
   - Somar os três em `custo_total`. Se algum não aparecer, somar o que houver e sinalizar a falta.
3. Geocodificar o `endereco` (Nominatim) → `lat`/`lng`. Conferir se o ponto cai no bairro esperado.
4. **Antes de escrever**, reportar em tabela: cada campo, o valor lido, e marcar `FALTOU` no que não achei.
5. Só depois do meu OK, inserir o objeto novo imediatamente antes do `];` que fecha o array, mantendo a indentação.

## Regras

- Não inventar valores. Campo não encontrado → string vazia / `null` + marcar como faltando no relatório.
- `id` único; se o anúncio tiver código, usar como base.
- Telefone sempre normalizado para `55DDD...` (sem espaços, parênteses ou traços).
- Site que bloqueia leitura (JS pesado / antibot): avisar e me pedir o texto colado do anúncio para parsear.
- `proposta`, `visita`, `visita_inicio` ficam vazios na ingestão — preencho manualmente.
