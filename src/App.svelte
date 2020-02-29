<main style="--theme-color: {bgColor.hex}">
    <div style="--inverted-color: {invertedColor}">
        <h1>Color this page</h1>
        {#if noResult && hasRun && query !== ''}
            <h2>No color found</h2>
        {:else if hasRun && query !== ''}
            <h2>{bgColor.name}</h2>
        {:else}
            <h2>Type in a color below</h2>
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
    getColors().then( res => {
        colors = res
    } )
    let bgColor = { hex: '#fff' }
    let noResult = false
    let hasRun = false
    let invertedColor = '#074ddc'
    let query
    $: if ( colors && query ) {
        findAndSetColors()
        hasRun = true
    }
    else if (query === '') {
    	bgColor.hex = '#fff'
		invertedColor = '#074ddc'
	}

    async function findAndSetColors ( name ) {
        await colors
        const bgColorArray = colors.filter( color => color.name.toLowerCase().includes( query.toLowerCase() ) )
        if ( bgColorArray.length >= 1 && query !== '' ) {
            noResult = false
            bgColor = bgColorArray[ 0 ]
            invertedColor = invertColor( bgColor.hex )
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

    h1, h2 {
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
    }
</style>