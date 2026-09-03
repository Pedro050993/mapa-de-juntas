# Mapa de Juntas — Controle de Tubulação

Controle de juntas soldadas: mapa, painel de avanço e relatórios FOR-266 (Ensaio
Visual de Solda) e FOR-053 (Líquido Penetrante).

## Stack

- HTML/CSS/JS puro, sem build — `index.html` é o aplicativo inteiro
- Supabase (banco compartilhado + realtime)
- Vercel (site estático)

Não há `package.json` de propósito: o aplicativo não tem dependências de build.
A única biblioteca externa é o `supabase-js`, carregado por CDN. O leitor/gravador
de `.xlsx` e o de `.zip` são próprios (`lib-xlsx.js` / `lib-zip.js`, embutidos).

## Dois modos, o mesmo arquivo

O módulo `DB` escolhe onde gravar:

| Situação | Onde grava | Quem enxerga |
|---|---|---|
| `index.html` aberto direto no computador | IndexedDB | só aquele navegador |
| Publicado (esta pasta, na Vercel) | Supabase | todos que abrem o link |

A troca é automática: se `window.MAPA_JUNTAS_SUPABASE` existir e o `supabase-js`
tiver carregado, usa o banco; senão cai no IndexedDB. A barra superior mostra
`⇅ compartilhado` ou `⌂ só neste navegador`.

## Banco

Projeto `ylojxyneplssoogvqcyt` (região sa-east-1).

```
mj_mapas    um registro por mapa/contrato
mj_dados    config, soldadores, overrides, logotipo e assinaturas (jsonb por chave)
mj_juntas   UMA LINHA POR JUNTA
```

Uma linha por junta é deliberado: dois inspetores lançando ensaios em juntas
diferentes nunca colidem, e cada gravação sobe só o que mudou.

### RLS

Liberada para a chave anônima — qualquer pessoa com o link lê **e escreve**.
Para exigir login depois, troque a policy `acesso_publico` por uma que valide
`auth.uid()` e adicione uma tela de login.

## Deploy

```sh
git init && git add . && git commit -m "Mapa de Juntas"
git remote add origin <repo>
git push -u origin main
```

Depois, na Vercel: importar o repositório, **Framework Preset: Other**, sem
build command e com o output na raiz. O `vercel.json` já cuida do resto.

## Levar um mapa existente para o online

O IndexedDB do arquivo local e o banco são separados; não há migração automática.

1. No arquivo local → Configurações → **Exportar backup (.json)**
2. No site → Configurações → **Restaurar backup** (cria um mapa novo)

## Modelos .xlsx

Ficam sempre no navegador de cada usuário, nunca no banco — são binários de
vários MB. Quem for exportar precisa importar os modelos uma vez, em
Configurações → Modelos Excel.
