<script>
    // Reactivity
	let count = 0;

    $: doubled = count * 2;

    $: console.log(`${count} was doubled;`); 

    $: if (count >= 10) {
	    alert('The count is too high!');
	    count = 0;
    }

	function increment() {
		count += 1;
	}

     // Reactivity - arrays
    let numbers = [1, 2];

    function addNumber() {
        numbers.push(numbers.length + 1);
        numbers = numbers

        // Or using spread: numbers = [...numbers, numbers.length + 1];
    }

    $: sum = numbers.reduce((total, currentNumber) => total + currentNumber, 0);

     // Lifecycle - onMount
    import { onMount } from 'svelte';
	import { paint } from './gradient.js';

    onMount(() => {
	const canvas = document.querySelector('canvas')
	const context = canvas.getContext('2d');
    
    // Without cleaning
    requestAnimationFrame(function loop(t) {
			requestAnimationFrame(loop);
			paint(context, t);
		});
	});

    // After cleaning
	// let frame = requestAnimationFrame(function loop(t) {
	// 	frame = requestAnimationFrame(loop);
	// 	paint(context, t);
	// });

	// return () => {
	// 	cancelAnimationFrame(frame);
	// };
    // });

    // Lifecycle - beforeUpdate and afterUpdate

    import Eliza from 'elizabot';
	import {
		beforeUpdate,
		afterUpdate
	} from 'svelte';

	let div;
    // let autoscroll = false;
    
	beforeUpdate(() => {
		// determine whether we should auto-scroll
		// once the DOM is updated...
    //     if (div) {
	// 	const scrollableDistance = div.scrollHeight - div.offsetHeight;
	// 	autoscroll = div.scrollTop > scrollableDistance - 20;
	// }
	});

	afterUpdate(() => {
		// ...the DOM is now in sync with the data
    //     if (autoscroll) {
	// 	div.scrollTo(0, div.scrollHeight);
	// }
	});

	const eliza = new Eliza();
	const pause = (ms) => new Promise((fulfil) => setTimeout(fulfil, ms));

	const typing = { author: 'eliza', text: '...' };

	let comments = [];

	async function handleKeydown(event) {
		if (event.key === 'Enter' && event.target.value) {
			const comment = {
				author: 'user',
				text: event.target.value
			};

			const reply = {
				author: 'eliza',
				text: eliza.transform(comment.text)
			};

			event.target.value = '';
			comments = [...comments, comment];

			await pause(200 * (1 + Math.random()));
			comments = [...comments, typing];

			await pause(500 * (1 + Math.random()));
			comments = [...comments, reply].filter(comment => comment !== typing);
		}
	}
    // Lifecycle - tick
    import { tick } from 'svelte';

    let text = `Select some text and hit the tab key to toggle uppercase. The selection is lost!`;

	async function handleKeydownTick(event) {
		if (event.key !== 'Tab') return;

		event.preventDefault();

		const { selectionStart, selectionEnd, value } = this;
		const selection = value.slice(selectionStart, selectionEnd);

		const replacement = /[a-z]/.test(selection)
			? selection.toUpperCase()
			: selection.toLowerCase();

		text =
			value.slice(0, selectionStart) +
			replacement +
			value.slice(selectionEnd);

        // await tick();
		// this.selectionStart = selectionStart;
		// this.selectionEnd = selectionEnd;
	}
    // ContextAPI

    

    import Canvas from './Canvas.svelte';
	import Square from './Square.svelte';

	// we use a seeded random number generator to get consistent jitter
	let seed = 1;

	function random() {
		seed *= 16807;
		seed %= 2147483647;
		return (seed - 1) / 2147483646;
	}

	function jitter(amount) {
		return amount * (random() - 0.5);
	}
</script>

<div class="container">
    <h1 class="display-1">Svelte In-depth</h1>
    <hr>

    <div class="container">
        <h1 class="display-6">Reactivity - assignments</h1>
        <button class="btn btn-primary mt-3 mb-3" on:click={increment}>
            Clicked {count}
            {count === 1 ? 'time' : 'times'}
        </button>

        <p>{count} doubled is {doubled}</p>
    </div>

    <hr>

    <div class="container">
        <h1 class="display-6">Reactivity - arrays</h1>

        <p>{numbers.join(' + ')} = {sum}</p>
        
        <button class="btn btn-primary" on:click={addNumber}>
            Add a number
        </button>
    </div>

    <hr>

    <div class="container">
        <h1 class="display-6">Lifecycle - onMount</h1>

        <canvas
            width={16}
            height={8}
        />
        
    </div>
    <hr>

    <div class="container">
        <h1 class="display-6">Lifecycle - beforeUpdate and afterUpdate</h1>
        <div class="eliza-container">
	        <div class="phone">
                <div class="chat" bind:this={div}>
                    <header>
                        <h1 class="eliza-header">Eliza</h1>

                        <article class="eliza">
                            <span>{eliza.getInitial()}</span>
                        </article>
                    </header>

                    {#each comments as comment}
                        <article class={comment.author}>
                            <span>{comment.text}</span>
                        </article>
                    {/each}
                </div>
		    <input on:keydown={handleKeydown} />
	        </div>
        </div>
    </div>

    <hr>
    <div class="container">
        <h1 class="display-6">Lifecycle - Tick</h1>
        <textarea
            value={text}
            on:keydown={handleKeydownTick}
        />
    </div>
    
    <hr>
    <div class="container mb-3">
        <h1 class="display-6">ContextAPI</h1>
        <div class="canvas-container">
            <Canvas width={800} height={1200}>
                {#each Array(12) as _, c}
                    {#each Array(22) as _, r}
                        <Square
                            x={180 + c * 40 + jitter(r * 2)}
                            y={180 + r * 40 + + jitter(r * 2)}
                            size={40}
                            rotate={jitter(r * 0.05)}
                        />
                    {/each}
                {/each}
            </Canvas>
        </div>
    </div>
</div>

<style>
    @import 'bootstrap/dist/css/bootstrap.min.css';

    canvas {
		position: relative;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
		background-color: #666;
		mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
		mask-size: 60vmin;
		-webkit-mask: url(svelte-logo-mask.svg) 50% 50% no-repeat;
		-webkit-mask-size: 60vmin;
	}

	.eliza-container {
		display: grid;
		place-items: center;
		height: 100%;
	}

	.phone {
		display: flex;
		flex-direction: column;
		width: 100%;
		height: 100%;
	}

	header {
		display: flex;
		flex-direction: column;
		height: 100%;
		padding: 4em 0 0 0;
		box-sizing: border-box;
	}

	.eliza-header {
		flex: 1;
		font-size: 1.4em;
		text-align: center;
	}

	.chat {
		height: 0;
		flex: 1 1 auto;
		padding: 0 1em;
		overflow-y: auto;
		scroll-behavior: smooth;
	}

	article {
		margin: 0 0 0.5em 0;
	}

	.user {
		text-align: right;
	}

	span {
		padding: 0.5em 1em;
		display: inline-block;
	}

	.eliza span {
		background-color: greenyellow;
		border-radius: 1em 1em 1em 0;
		color: var(--fg-1);
	}

	.user span {
		background-color: #0074d9;
		color: white;
		border-radius: 1em 1em 0 1em;
		word-break: break-all;
	}

	input {
		margin: 0.5em 1em 1em 1em;
	}

	@media (min-width: 400px) {
		.phone {
			background: white;
			position: relative;
			font-size: min(2.5vh, 1rem);
			width: auto;
			height: 36em;
			aspect-ratio: 9 / 16;
			border: 0.2em solid #222;
			border-radius: 1em;
			box-sizing: border-box;
			filter: drop-shadow(1px 1px 0px #222) drop-shadow(2px 2px 0px #222) drop-shadow(3px 3px 0px #222)
		}

		.phone::after {
			position: absolute;
			content: '';
			background: #222;
			width: 60%;
			height: 1em;
			left: 20%;
			top: 0;
			border-radius: 0 0 0.5em 0.5em
		}
	}

	@media (prefers-reduced-motion) {
		.chat {
			scroll-behavior: auto;
		}
	}

    textarea {
		width: 100%;
		height: 100%;
		resize: none;
	}

    .canvas-container {
		height: 100%;
        width: 30%;
		aspect-ratio: 2 / 3;
		margin: 0 auto;
		background: rgb(224, 219, 213);
		filter: drop-shadow(0.5em 0.5em 1em rgba(0, 0, 0, 0.1));
	}
</style>