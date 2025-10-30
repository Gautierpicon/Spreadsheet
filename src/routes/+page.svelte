<script lang="ts">
    const rows = 10
    const cols = 10
    const letters = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'

    const data = $state([
        [{ value: 'Item'}, { value: 'Price' }, { value: 'Amount' }, { value: 'Total' }],
        [{ value: '🍌'}, { value: '1' }, { value: '4' }, { value: '=MULTIPLY(B2,C2)' }],
        [{ value: '🍏'}, { value: '2' }, { value: '2' }, { value: '=MULTIPLY(B3,C3)' }],
        [{ value: ''}, { value: '' }, { value: 'Total' }, { value: '=SUM(D2,D3)' }],
    ])

    let selectedCell = $state()
    let editedCell = $state()

    function parseValue(value: string): string | number {
        if (value.startsWith('=')) {
            const { operation, cells } = parseFormula(value)

            const values = cells.map(({ row, col }) => {
                const value = data[row][col].value
                if (value.startsWith('=')) {
                    return +parseValue(value)
                }
                return +value
            })

            return values.reduce((total, value) => {
                if (operation === 'SUM') return total + value
                if (operation === 'MULTIPLY') return total * value
                return total 
            }, operation === 'MULTIPLY' ? 1 : 0)
        }
        return value
    }

    function parseFormula(value: string) {
        const [a, b] = value.split('(')
        const operation = a.split('=')[1]
        const cells = b.replace(')', '').split(',')
        return { operation, cells: cells.map(cellNameToIndex) }
    }

    function cellNameToIndex(value: string) {
        const [col, ...row] = value.split('')
        return {row: +row.join('') - 1, col: letters.indexOf(col) }
    }

    function update(e: Event) {
        const { value, parentElement } = e.target as HTMLInputElement
        const { row, col } = cellNameToIndex(parentElement!.dataset.cell!)

        if (data[row]) {
            if (data[row][col]) {
                data[row][col].value = value
            } else {
                data[row][col] = { value }
            }
        } else {
            data[row] = []
            data[row][col] = { value }
        }
    }
</script>

<div class="flex justify-center items-center min-h-screen">
    <table class="max-w-[800px] bg-gray-100 rounded-lg overflow-hidden border-collapse">
        <thead>
            <tr>
                <th class="border border-gray-300 bg-gray-200"></th>
                {#each Array(rows) as _, i}
                    <th class="border border-gray-300 bg-gray-200">{letters[i]}</th>
                {/each}
            </tr>
        </thead>

        <tbody>
            {#each Array(rows) as _, i}
                <tr>
                    <th class="border border-gray-300 bg-gray-200 w-10">{i + 1}</th>
                    {#each Array(cols) as _, j}
                        {@const cell = `${letters[j]}${i + 1}`}
                        {@const value = data[i]?.[j]?.value}
                        {@const parsedValue = value ? parseValue(value) : ''}
                        {@const selected = selectedCell === cell}
                        {@const editing = editedCell === cell}
                        <td 
                            class="w-40 h-10 font-semibold text-center border border-gray-300 bg-white {selected ? 'outline-2 outline-blue-500' : ''}"
                            data-cell={cell} 
                            onclick={() => {
                                if (selected) return
                                selectedCell = cell
                                editedCell = null
                            }} 
                            ondblclick={() => editedCell = cell }
                        >
                            {#if editing}
                                <!-- svelte-ignore a11y_autofocus -->
                                <input 
                                    type="text" 
                                    {value} 
                                    oninput={update} 
                                    autofocus 
                                    class="h-full w-full p-0 bg-transparent outline-none text-center font-semibold"
                                >
                            {:else}
                                <span>{parsedValue}</span>
                            {/if}
                        </td>
                    {/each}
                </tr>
            {/each}
        </tbody>
    </table>
</div>