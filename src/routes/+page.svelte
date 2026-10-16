<script lang="ts">
	import ArrowLeftRight from '$lib/icons/ArrowLeftRight.svelte';
	import { example_pgn } from '$lib/index';
	import { onMount } from 'svelte';
	const WIDTH = 200;
	const HEIGHT = 200;

	onMount(() => {
		document.title = 'IMG PGN';
		load_image();
	});

	let canvas_element = $state<HTMLCanvasElement>();
	let files = $state<FileList | null | undefined>(null);

	let final_pgn = $state(example_pgn);
	let image_size_mono = $state(10);
	let image_size = $derived({
		height: image_size_mono,
		width: image_size_mono
	});

	const pretext = $derived(
		`[Event "${image_size_mono}x${image_size_mono}"]\n[Variant "From Position"]\n[FEN "7r/6pk/6r1/8/8/6R1/6PK/RrRrRr1R w - - 0 1"]\n`
	);

	class EncodeSpecial {
		private g_rook_white;
		private g_rook_black;

		constructor() {
			this.g_rook_white = 3;
			this.g_rook_black = 6;
		}

		write_same(move: number) {
			if (move % 2) {
				if (this.g_rook_white == 4) {
					this.g_rook_white = 3;
					return 'Rgg3';
				} else {
					this.g_rook_white = 4;
					return 'Rgg4';
				}
			} else {
				if (this.g_rook_black == 6) {
					this.g_rook_black = 5;
					return 'Rgg5';
				} else {
					this.g_rook_black = 6;
					return 'Rgg6';
				}
			}
		}
	}

	function splitter(lval: number, rval: number, size: number) {
		if (size > 8) {
			rval = size - 8;
			lval = 8;
		} else {
			lval = size;
		}
		return [lval, rval];
	}

	function generate_moves(pixels: Array<number>) {
		let a_rook = 1;
		let b_rook = 1;
		let c_rook = 1;
		let d_rook = 1;
		let e_rook = 1;
		let f_rook = 1;
		const encoder = new EncodeSpecial();

		let output = [pretext];
		let counter = 0;
		// pixels move left to right, top to bottom
		for (let x = 0; x < pixels.length; x += 3) {
			counter += 1;
			let pgn = `${counter}. `;
			const red = Math.floor(pixels[x] * (16 / 255)) + 1;
			const green = Math.floor(pixels[x + 1] * (16 / 255)) + 1;
			const blue = Math.floor(pixels[x + 2] * (16 / 255)) + 1;

			// console.log(pixels[x], pixels[x + 1], pixels[x + 2]);

			let ia_rook = a_rook;
			let ib_rook = b_rook;
			let ic_rook = c_rook;
			let id_rook = d_rook;
			let ie_rook = e_rook;
			let if_rook = f_rook;

			[a_rook, b_rook] = splitter(a_rook, b_rook, red);
			[c_rook, d_rook] = splitter(c_rook, d_rook, green);
			[e_rook, f_rook] = splitter(a_rook, b_rook, blue);

			if (a_rook == ia_rook) {
				pgn += encoder.write_same(1) + ' ';
			} else {
				pgn += `Raa${a_rook} `;
			}

			if (b_rook == ib_rook) {
				pgn += encoder.write_same(2);
			} else {
				pgn += `Rbb${b_rook}`;
			}

			counter += 1;
			pgn += ` ${counter}. `;

			if (c_rook == ic_rook) {
				pgn += encoder.write_same(1) + ' ';
			} else {
				pgn += `Rcc${c_rook} `;
			}

			if (d_rook == id_rook) {
				pgn += encoder.write_same(2);
			} else {
				pgn += `Rdd${d_rook}`;
			}

			counter += 1;
			pgn += ` ${counter}. `;

			if (e_rook == ie_rook) {
				pgn += encoder.write_same(1) + ' ';
			} else {
				pgn += `Ree${e_rook} `;
			}

			if (f_rook == if_rook) {
				pgn += encoder.write_same(2);
			} else {
				pgn += `Rff${f_rook}`;
			}

			output.push(pgn);
		}
		return output.join(' ');
	}

	function read_image(pgn: string) {
		const segments = pgn.split('\n');
		const index = (() => {
			let i = 0;
			for (let x = 0; x < segments.length; x++) {
				// if (segments[x].includes('Event')) {
				// 	const m = segments[x].match(/Event "(\d+)x(\d+)"/);
				// 	try {
				// 		if (m) image_size_mono = parseInt(m[1]);
				// 	} catch {}
				// }
				if (segments[x].match(/^\s*1. /)) i = x;
			}
			return i;
		})() as number;
		// console.log(segments[index]);
		// const data_blocks = segments.slice(index, segments.length);
		const data = segments[index];
		// const data = data_blocks.join(' ');
		let objs = data.matchAll(/\d+\. \S+ \S+/gm);
		let count = 0;
		objs.forEach((e) => {
			count++;
		});
		image_size_mono = Math.sqrt(count / 3);

		let a_rook = 1;
		let b_rook = 1;
		let c_rook = 1;
		let d_rook = 1;
		let e_rook = 1;
		let f_rook = 1;

		const pixels: Array<number> = [];
		let red = 0;
		let green = 0;
		let blue = 0;

		const ratio = Math.floor(255 / 16);
		objs = data.matchAll(/\d+\. \S+ \S+/gm);
		objs.forEach((v, index) => {
			const mod = index % 3;
			const _match = v[0].match(/(\S+) (\S+)$/);
			if (_match == null) return;
			const left_value = _match[1];
			const right_value = _match[2];

			switch (mod) {
				case 0: {
					if (left_value.includes('g')) {
						red += a_rook - 1;
					} else {
						const val = parseInt(left_value.slice(left_value.length - 1));
						red += val - 1;
						a_rook = val;
					}

					if (right_value.includes('g')) {
						red += b_rook - 1;
					} else {
						const val = parseInt(right_value.slice(right_value.length - 1));
						red += val - 1;
						b_rook = val;
					}

					break;
				}

				case 1: {
					if (left_value.includes('g')) {
						green += c_rook - 1;
					} else {
						const val = parseInt(left_value.slice(left_value.length - 1));
						green += val - 1;
						c_rook = val;
					}

					if (right_value.includes('g')) {
						green += d_rook - 1;
					} else {
						const val = parseInt(right_value.slice(right_value.length - 1));
						green += val - 1;
						d_rook = val;
					}

					break;
				}

				default: {
					if (left_value.includes('g')) {
						blue += e_rook - 1;
					} else {
						const val = parseInt(left_value.slice(left_value.length - 1));
						blue += val - 1;
						e_rook = val;
					}

					if (right_value.includes('g')) {
						blue += f_rook - 1;
					} else {
						const val = parseInt(right_value.slice(right_value.length - 1));
						blue += val - 1;
						f_rook = val;
					}

					red *= ratio;
					green *= ratio;
					blue *= ratio;
					pixels.push(red, green, blue, 255);

					red = 0;
					green = 0;
					blue = 0;

					break;
				}
			}
			// console.log(right_value, left_value);
		});
		return pixels;
	}

	function load_image() {
		if (!canvas_element) return;
		const pixels = read_image(final_pgn);
		const ctx = canvas_element.getContext('2d');
		if (!ctx) return;

		const temp = document.createElement('canvas');
		temp.width = image_size.width;
		temp.height = image_size.height;

		const temp_ctx = temp.getContext('2d');
		if (!temp_ctx) return;

		const image_data = temp_ctx.createImageData(image_size.width, image_size.height);
		image_data.data.set(pixels);
		temp_ctx.putImageData(image_data, 0, 0);

		ctx.clearRect(0, 0, canvas_element.width, canvas_element.height);
		ctx.drawImage(temp, 0, 0, canvas_element.width, canvas_element.height);
	}

	$effect(() => {
		if (!files || !canvas_element) return;
		const ctx = canvas_element.getContext('2d');
		if (!ctx) return;
		if (files.length == 0) return;

		const img = new Image();
		img.src = URL.createObjectURL(files[0]);
		// canvas_element.height = HEIGHT;
		// canvas_element.width = WIDTH;
		img.onload = function () {
			if (!canvas_element) return;
			// const scale = Math.min(canvas_element.width / img.width, canvas_element.height / img.height);
			// const width = img.width * scale;
			// const height = img.height * scale;
			// const x = (canvas_element.width - width) / 2;
			// const y = (canvas_element.height - height) / 2;
			// ctx.clearRect(0, 0, canvas_element.width, canvas_element.height);
			ctx.drawImage(img, x, y, width, height);

			// format is RGBA :(
			const image_data = ctx.getImageData(0, 0, image_size.width, image_size.height);
			const clean_pixels = image_data.data.filter(
				(_, index) => (index + 1) % 4 !== 0
			) as any as Array<number>;
			final_pgn = generate_moves(clean_pixels);

			load_image();
		};
	});
</script>

<div class="glass flex flex-col items-center justify-center rounded-lg px-3 py-5">
	<h1 class="text-5xl font-bold">Image to PGN</h1>
	<p class="italic">Convert images to chess PGN and back!</p>

	<div class="mt-4 flex flex-row items-center justify-center gap-3">
		<div class="flex w-auto flex-col gap-2">
			<canvas
				onclick={() => {
					(document.querySelector('#file_input') as any).click();
				}}
				bind:this={canvas_element}
				width="200"
				height="200"
				class="glass cursor-pointer rounded-sm"
			>
				IMAGE CANVAS PREVIEW
			</canvas>
		</div>
		<ArrowLeftRight />
		<div class="flex flex-col gap-2">
			<textarea
				style:width={`${WIDTH}px`}
				style:height={`${HEIGHT}px`}
				class="resize-none overflow-y-scroll border-1 whitespace-pre-line"
				contenteditable="true"
				bind:value={final_pgn}></textarea>
		</div>
	</div>
	<div class="mt-2 flex w-full flex-row justify-between">
		<div style:width={`${WIDTH}px`}>
			<label for="file_input" class="w-fit rounded bg-green-400 px-3 py-1"> Select Image </label>
			<input id="file_input" bind:files type="file" hidden />
		</div>
		<div class="flex flex-col gap-2" style:width={`${WIDTH}px`}>
			<button class="w-fit rounded bg-blue-400 px-3 py-1" onclick={load_image}>Load From PGN</button
			>
			<button
				class="w-fit rounded bg-amber-400 px-3 py-1"
				onclick={() => {
					navigator.clipboard.writeText(final_pgn);
				}}
			>
				Copy PGN to Clipboard
			</button>
		</div>
	</div>

	<fieldset class="flex max-w-sm flex-col rounded border-1 px-3 py-2">
		<label for="res_select"> Resolution: {image_size_mono}px </label>
		<input type="number" min="1" max="500" bind:value={image_size_mono} />
		<input id="res_select" type="range" min="1" max="500" bind:value={image_size_mono} />
		<div>
			PGN Length: {image_size_mono * image_size_mono * 3}
			<br />
			File Size:{#if final_pgn}
				~{Math.ceil(final_pgn.length / 1000)}KiB{/if}
		</div>
	</fieldset>
</div>
