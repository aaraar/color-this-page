<main style="--theme-color: {bgColor.hex}">
    <div style="--inverted-color: {invertedColor}">
        <h1>Color this page</h1>
        {#if noResult && hasRun && query !== ''}
            <h2>Ah bummer</h2>
            <h3>No colors found</h3>
        {:else if hasRun && query !== ''}
            <h2>{bgColor.name}</h2>
            <h3>{bgColorArray.length} colors found</h3>
        {:else}
            <h2>Type in a color below</h2>
            <h3>And the page will cycle through the results</h3>
        {/if}
        {#await colors}
            <p>Retrieving Color List...</p>
        {:then colors}
            <label>
                <input type="text" placeholder="color" bind:value={query}/>
            </label>
        {/await}
    </div>
</main>

<script>
    let colors
    if ( localStorage.getItem( 'colors' ) ) {
        colors = JSON.parse( localStorage.getItem( 'colors' ) )
    } else {
        getColors().then( res => {
            colors = res
            localStorage.setItem( 'colors', JSON.stringify( colors ) )
        } )
    }
    let bgColor = { hex: '#fff' }
    let noResult = false
    let hasRun = false
    let interval
    let invertedColor = '#074ddc'
    let bgColorArray
    let query = ''
    $: if ( query === '' ) {
        bgColor.hex = '#fff'
        invertedColor = '#074ddc'
        findAndSetColors()
    } else if ( colors ) {
        findAndSetColors()
    }

    async function findAndSetColors ( name ) {
        window.clearInterval( interval )
        if ( query !== '' ) {
            await colors
            bgColorArray = colors.filter( color => color.name.toLowerCase().includes( query.toLowerCase() ) )
            if ( bgColorArray.length >= 1 ) {
                let intervalCounter = 0
                noResult = false
                hasRun = true
                bgColor = bgColorArray[ intervalCounter ]
                invertedColor = invertColor( bgColor.hex )
                interval = window.setInterval( () => {
                    if ( intervalCounter < bgColorArray.length - 1 ) {
                        intervalCounter++
                    } else {
                        intervalCounter = 0
                    }
                    bgColor = bgColorArray[ intervalCounter ]
                    invertedColor = invertColor( bgColor.hex )
                }, 1000 )
            }
        } else {
            noResult = true
        }

    }

    function invertColor ( hex = '#000' ) {
        if ( hex.indexOf( '#' ) === 0 ) {
            hex = hex.slice( 1 )
        }
        // convert 3-digit hex to 6-digits.
        if ( hex.length === 3 ) {
            hex = hex[ 0 ] + hex[ 0 ] + hex[ 1 ] + hex[ 1 ] + hex[ 2 ] + hex[ 2 ]
        }
        if ( hex.length !== 6 ) {
            throw new Error( 'Invalid HEX color.' )
        }
        // invert color components
        var r = (255 - parseInt( hex.slice( 0, 2 ), 16 )).toString( 16 ),
                g = (255 - parseInt( hex.slice( 2, 4 ), 16 )).toString( 16 ),
                b = (255 - parseInt( hex.slice( 4, 6 ), 16 )).toString( 16 )
        // pad each with zeros and return
        return '#' + padZero( r ) + padZero( g ) + padZero( b )
    }

    function padZero ( str, len ) {
        len = len || 2
        var zeros = new Array( len ).join( '0' )
        return (zeros + str).slice( -len )
    }

    async function getColors () {
        const res = await fetch( `https://api.color.pizza/v1/` )
        const resObj = await res.json()

        if ( res.ok ) {
            return resObj.colors
        } else {
            throw new Error( colors )
        }
    }
</script>
<style>
    main {
        text-align: center;
        position: absolute;
        display: flex;
        justify-content: center;
        align-items: center;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        margin: 0;
        background-color: var(--theme-color);
        transition: background-color 0.1s ease-in-out;
    }

    div {
        margin-bottom: 2em;
    }

    h1 {
        text-transform: uppercase;
        font-size: 4em;
        font-weight: 100;
    }

    h1, h2, h3 {
        color: var(--inverted-color);
    }

    @media (max-width: 640px) {
        main {
            max-width: none;
        }

        h1 {
            font-size: 2em;
        }

        h2 {
            font-size: 1em;
        }
        h3 {
            font-size: 0.7em;
        }
    }
</style>