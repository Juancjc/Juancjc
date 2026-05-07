# Regra do botão "Novo DAE"

Este componente exibe o botão **Novo DAE** quando o DAE atual está vencido e ainda pode ser regerado.

## Condições para exibir o botão

O botão **Novo DAE** aparece somente quando todas as condições abaixo forem verdadeiras:

- O item possui `id`;
- O item possui `validade_dae`;
- O tipo do arremate é permitido:
  - `Entrada`
  - `Taxa Administrativa`
  - `Arremate`
  - `Comissao`
  - `Comissão`
- O `status_id` é `18`;
- A data de validade do DAE já venceu;
- O campo `novo_dae_gerado` é falso.

## Código principal

```js
const STATUS_PAGO_ID = 17
const STATUS_AGUARDANDO_ID = 18

function normalizarTexto(valor) {
  return String(valor ?? "")
    .trim()
    .toLowerCase()
    .normalize("NFD")
    .replace(/[\u0300-\u036f]/g, "")
}

const tiposPermitemNovoDae = [
  "entrada",
  "taxa administrativa",
  "arremate",
  "comissao",
]

const novoDaeJaGerado = computed(() => {
  return (
    props.item?.novo_dae_gerado === true ||
    props.item?.novo_dae_gerado === 1 ||
    props.item?.novo_dae_gerado === "1" ||
    props.item?.novo_dae_gerado === "true"
  )
})

const daeVencido = computed(() => {
  if (!props.item?.validade_dae) return false

  const validade = dayjs(props.item.validade_dae)

  return validade.isValid() && validade.isBefore(dayjs(), "day")
})

const podeGerarNovoDae = computed(() => {
  const item = props.item

  if (!item?.id) return false
  if (!item?.validade_dae) return false

  const tipo = normalizarTexto(item.tipo_arremate)
  const statusId = Number(item.status_id)

  return (
    tiposPermitemNovoDae.includes(tipo) &&
    statusId === STATUS_AGUARDANDO_ID &&
    daeVencido.value &&
    !novoDaeJaGerado.value
  )
})
```

## Botão no template

```vue
<button
  v-if="podeGerarNovoDae"
  type="button"
  class="btn btn-warning btn-sm px-2 py-1 botao-acao"
  :disabled="gerandoNovoDae"
  @click="gerarNovoDae"
>
  <i
    v-if="!gerandoNovoDae"
    class="far fa-file-invoice-dollar small pe-1"
  ></i>

  <i
    v-else
    class="fa-solid fa-spinner fa-spin small pe-1"
  ></i>

  <span class="small">Novo DAE</span>
</button>
```

## Exemplo de item que deve exibir o botão

```json
{
  "id": 71,
  "status_id": 18,
  "tipo_arremate": "Taxa Administrativa",
  "validade_dae": "2026-03-07T05:00:00.000000Z",
  "novo_dae_gerado": false
}
```

Neste exemplo, o botão aparece porque:

- O status é `18`;
- O tipo é `Taxa Administrativa`;
- O DAE está vencido;
- Ainda não foi gerado um novo DAE.
