<script>
    const normalParam = 44;

    function someFunction(a, b, c) {
        const value = 'Extract'
        const next = `Interpolate ${a} ${b}`
        return next + c + value
    }

    async function someFunctionAsync() {
        const res = await fetch("https://jsonplaceholder.typicode.com/todos/1")
        const json = await res.json()
        json.name = 'Extracted name'
        return json
    }

    export const arrow = (msg) => {
        alert(`This page says: ${msg}`)
    }

    const collection = { members: [1] };
    const someJSEven = 34;
    const obj = $derived({
        property: {
            ["non-extracted text"]: { ["Extracted text"]: 42 },
        },
    });
    const p = {
        id: 23,
        name: 'foo',
    }

    const derived = $derived.by(() => {
        const f = 'Extract this'
        if (!f) return
        let e = {}
        if (f == 'something else') {
            let d = 'Variable'
            d = d + 'Add to Variable'
            return d
        } else if (f == 'Check extracted') {
            return f
        } else {
            for (const q of [1,2,3]) {
                e[`${q}/${collection.members[0]}`] = {...p, name: 'That'}
            }
            e.default = [f, 'And this', e]
        }
        return {
            ['And this']: f,
            butNotThis: 'Okay?',
            e
        }
    })
</script>

<div>{definition('foo', 'Bar')}</div>

<p>
    This is a very {obj.property["non-extracted text"]["Extracted text"]}
    Complicated
    <i class="not-extracted" title="Extracted"
        >and even <b><u>depply</u> nested {`with ${someJSEven}` + "foo"}</b> content</i
    >
    With
    <!-- foo bar -->
    {#if someFunction("Extracted Text", normalParam, [/* @wc-include */ "extracted anyway"])}
        Conditionals,
        {#each collection.members as member}
            Loops and {member}
            <!-- What not -->
            {#await someFunctionAsync(a) then json}
                <b>{json.title} other blocks</b>
            {/await}
            Supported
        {/each}
    {/if}
</p>
<!-- @wc-ignore -->
But ignore me
