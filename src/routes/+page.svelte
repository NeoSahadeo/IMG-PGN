<script lang="ts">
	import { example_pgn } from '$lib';
	let canvas_element = $state<HTMLCanvasElement>();
	let files = $state<FileList | null | undefined>(null);

	let final_pgn = $state('');
	let image_size_mono = 25;
	let image_size = $state({
		height: image_size_mono,
		width: image_size_mono
	});

	const pretext = `[Variant "From Position"]<br>
	[FEN "7r/6pk/6r1/8/8/6R1/6PK/RrRrRr1R w - - 0 1"]<br>
	`;

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
		final_pgn = output.join(' ');
	}

	function read_image(pgn: string) {
		const segments = pgn.split('\n');
		const index = (() => {
			for (let x = 0; x < segments.length; x++) if (segments[x].match(/^\s*1. /)) return x;
		})() as number;
		const data = segments[index];
		const objs = data.matchAll(/\d+\. \S+ \S+/gm);

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
		const pixels = read_image(example_pgn);
		// console.log(pixels);
		const ctx = canvas_element.getContext('2d');
		if (!ctx) return;

		canvas_element.height = image_size.height;
		canvas_element.width = image_size.width;

		const image_data = ctx.createImageData(image_size.width, image_size.height);
		image_data.data.set(pixels);

		ctx.putImageData(image_data, 0, 0);
	}

	$effect(() => {
		if (!files || !canvas_element) return;
		const ctx = canvas_element.getContext('2d');
		if (!ctx) return;
		if (files.length == 0) return;

		const img = new Image();
		img.src = URL.createObjectURL(files[0]);
		canvas_element.height = image_size.height;
		canvas_element.width = image_size.width;
		img.onload = function () {
			ctx.drawImage(img, 0, 0, image_size.width, image_size.height);

			// format is RGBA :(
			const image_data = ctx.getImageData(0, 0, image_size.width, image_size.height);
			const clean_pixels = image_data.data.filter(
				(_, index) => (index + 1) % 4 !== 0
			) as any as Array<number>;
			generate_moves(clean_pixels);

			ctx.clearRect(0, 0, image_size.width, image_size.height);
			ctx.drawImage(img, 0, 0, image_size.width, image_size.height);
		};
	});
</script>

<input bind:files type="file" />
<button onclick={load_image}>Load Image</button>
<canvas bind:this={canvas_element} class="bg-black"> IMAGE CANVAS PREVIEW </canvas>
<div>
	<h2>PGN:</h2>
	<span class="max-h-20 overflow-y-scroll">
		{@html final_pgn}
	</span>
</div>
