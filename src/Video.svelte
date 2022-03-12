<script>

    // Selected files
    let files = 0;
    let src;
	let src2;
    $: poster = src + "#t=10";
	$: poster2 = src2 + "#t=10";

    $: file_name = files ? files[0].name.split('.')[0] : 0;

	//const timingObject = new TimingObject();

	//querySelector(<webcomponent1>).timingSrc = timingObject;
	//querySelector(<webcomponent2>).timingSrc = timingObject;

	// These values are bound to properties of the video
	let time = 0;
	let duration;
	let paused = true;

	let showControls = true;
	let showControlsTimeout;

	// Used to track time of last mouse down event
	let lastMouseDown;

	function handleMove(e) {
		// Make the controls visible, but fade out after
		// 2.5 seconds of inactivity
		clearTimeout(showControlsTimeout);
		showControlsTimeout = setTimeout(() => showControls = false, 2500);
		showControls = true;

		if (!duration) return; // video not loaded yet
		if (e.type !== 'touchmove' && !(e.buttons & 1)) return; // mouse not down

		const clientX = e.type === 'touchmove' ? e.touches[0].clientX : e.clientX;
		const { left, right } = this.getBoundingClientRect();
		time = duration * (clientX - left) / (right - left);
	}

	// we can't rely on the built-in click event, because it fires
	// after a drag — we have to listen for clicks ourselves
	function handleMousedown(e) {
		lastMouseDown = new Date();
	}

	function handleMouseup(e) {
		if (new Date() - lastMouseDown < 300) {
			if (paused) e.target.play();
			else e.target.pause();
		}
	}

	function format(seconds) {
		if (isNaN(seconds)) return '...';

		const minutes = Math.floor(seconds / 60);
		seconds = Math.floor(seconds % 60);
		if (seconds < 10) seconds = '0' + seconds;

		return `${minutes}:${seconds}`;
	}

    function loadFile(e) {
		src = URL.createObjectURL(e.target.files[0]);
	}

	// function loadFile2(e) {
	// 	src2 = URL.createObjectURL(e.target.files[0]);
	// }
</script>

<h1>Multi-modal video analysis toolkit</h1>

<input on:change={loadFile} type = "file" accept="video/*" bind:files>
<!-- <input on:change={loadFile2} type = "file" accept="video/*" bind:files> -->

{#if files && files[0]}
	<p>
		Showing the video of { file_name }
	</p>
{/if}


<div class = "container">
	
	<video
        preload='auto'
		{src}
		on:mousemove={handleMove}
		on:touchmove|preventDefault={handleMove}
		on:mousedown={handleMousedown}
		on:mouseup={handleMouseup}
		bind:currentTime={time}
		bind:duration
		bind:paused>
		<track kind="captions">
	</video>


	<!-- <video
		preload='auto'
		{src2}
		
		>
		<track kind="captions">
	</video> -->
	<div class="nested"> 
		<div>Sub Item 1</div>
		<div>Sub Item 2</div>
		<div>Sub Item 3</div>
	</div>
	


	<div class="controls" style="opacity: {duration && showControls ? 1 : 0}">
		<progress value="{(time / duration) || 0}"/>
		<div class="info">
			<span class="time">{format(time)}</span>
            <span>{file_name}</span>
			<span>click anywhere to {paused ? 'play' : 'pause'} / drag to seek</span>
			<span class="time">{format(duration)}</span>
		</div>

	</div>


</div>

<style>
	div {
		position: relative;
		
	}

	.controls {
		position: absolute;
		top: 0;
		width: 100%;
		transition: opacity 1s;
	}

	.info {
		display: flex;
		width: 100%;
		justify-content: space-between;
	}

	span {
		padding: 0.2em 0.5em;
		color: white;
		text-shadow: 0 0 8px black;
		font-size: 1.4em;
		opacity: 0.7;
	}

	.time {
		width: 3em;
	}

	.time:last-child { text-align: right }

	progress {
		display: block;
		width: 100%;
		height: 10px;
		-webkit-appearance: none;
		appearance: none;
	}

	progress::-webkit-progress-bar {
		background-color: rgba(0,0,0,0.2);
	}

	progress::-webkit-progress-value {
		background-color: rgba(255,255,255,0.6);
	}

	video {
		width: 100%; 
	}

	.container {
		display: grid;
		grid-template-columns:repeat(2, 3fr 1fr);
		grid-gap: 1em;
	}

	.container > video{
		background:#eee;
		/* padding:1em; */
		border: rgb(223, 62, 62) 1px solid;
	}
	.nested {
		display:grid;
		/* grid-template-columns:repeat(1, 1fr); */
	}
	
	.nested > div{
		/* background:#daa; */
		/* padding:1em; */
		border: rgb(223, 62, 62) 1px solid;
	}
</style>
