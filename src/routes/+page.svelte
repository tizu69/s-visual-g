<script lang="ts">
	import { localStorageStore } from '@skeletonlabs/skeleton';

	const s = localStorageStore('svgInput', {
		path: '',
		size: { w: 16, h: 16 }
	});

	function parseSvgPath(_svgPath: string): string[][] {
		let svgPath = _svgPath;

		// eval (using new Function()) all {}s in the string and replace the result
		svgPath = svgPath.replace(/{([^{}]+)}/g, (_, p1) => {
			return new Function(`return (width, height) => (${p1})`)()($s.size.w, $s.size.h);
		});

		const commands = svgPath.match(/([A-Za-z])\s?[0-9\s\.,-]*/g);
		if (!commands) return [];

		return commands.map((command) => [command[0], ...command.slice(1).split(/[,\s]+/)].filter(Boolean));
	}
	function stringifySvgPath(commands: string[][]): string {
		return commands
			.map(
				(command) =>
					command[0] + command.slice(1, (pathCommands[command[0].toUpperCase()]?.a?.length ?? 0) + 1).join(',')
			)
			.join('')
			.trim();
	}

	const pathCommands: Record<string, { a: string[]; d: string }> = {
		M: { a: ['x', 'y'], d: 'Move cursor' },
		L: { a: ['x', 'y'], d: 'Draw a line' },
		H: { a: ['x'], d: 'Draw a horizontal line' },
		V: { a: ['y'], d: 'Draw a vertical line' },
		C: { a: ['x1', 'y1', 'x2', 'y2', 'x', 'y'], d: 'Draw a cubic bezier curve' },
		S: { a: ['x1', 'y1', 'x2', 'y2', 'x', 'y'], d: 'Draw a smooth cubic bezier curve' },
		Q: { a: ['x1', 'y1', 'x', 'y'], d: 'Draw a quadratic bezier curve' },
		T: { a: ['x1', 'y1', 'x', 'y'], d: 'Draw a smooth quadratic bezier curve' },
		A: { a: ['rx', 'ry', 'angle', 'largeF', 'sweepF', 'x', 'y'], d: 'Draw an arc' },
		Z: { a: [], d: 'Draw to the start of the current subpath' }
	};

	/** parsed path */
	let parth: string[][] = [];
	s.subscribe((v) => (parth = parseSvgPath(v.path)));

	let showoff: { x: number; y: number } | null = null;
</script>

<div class="p-3 flex flex-col gap-2">
	<h1 class="h1 text-center">s-visual-g</h1>

	<textarea
		class="textarea min-h-48 mt-4 font-mono"
		bind:value={$s.path}
		placeholder="SVG path"
		on:click={() => ($s.path = stringifySvgPath(parth))}
	/>

	<div class="flex gap-2">
		<div class="input-group input-group-divider grid-cols-[1fr_auto]">
			<input type="number" placeholder="16" bind:value={$s.size.w} />
			<div class="input-group-shim">w</div>
		</div>

		<div class="input-group input-group-divider grid-cols-[1fr_auto]">
			<input type="number" placeholder="16" bind:value={$s.size.h} />
			<div class="input-group-shim">h</div>
		</div>
	</div>

	<hr class="mx-auto w-16 my-4" />

	<div class="flex flex-col gap-1">
		{#each parth as command, i}
			<div class="card p-2 {command[0] == 'M' && 'mt-2'} first:mt-0">
				<div class="flex gap-2 items-center">
					<select class="select" bind:value={command[0]}>
						<optgroup label="Absolute">
							{#each Object.entries(pathCommands) as [command, data]}
								<option value={command}>{data.d} (abs)</option>
							{/each}
						</optgroup>

						<optgroup label="Relative">
							{#each Object.entries(pathCommands) as [command, data]}
								<option value={command.toLowerCase()}>{data.d}</option>
							{/each}
						</optgroup>
					</select>

					<div class="flex gap-1 ml-auto">
						{#if command[0] == 'M'}
							<button
								on:click={() => {
									if (showoff && showoff.x === +command[1] && showoff.y === +command[2]) return (showoff = null);

									showoff = { x: +command[1], y: +command[2] };
								}}
								class="btn-icon btn-icon-sm {showoff && showoff.x === +command[1] && showoff.y === +command[2]
									? 'text-white bg-[#0000FF]'
									: 'variant-filled-primary'}"
							>
								?
							</button>
						{/if}

						<button
							on:click={() => (parth = [...parth.slice(0, i + 1), ['M', '0', '0'], ...parth.slice(i + 1)])}
							class="btn-icon btn-icon-sm variant-soft-primary"
						>
							+
						</button>
						<button
							on:click={() => (parth = [...parth.slice(0, i), ...parth.slice(i + 1)])}
							class="btn-icon btn-icon-sm variant-soft-primary"
						>
							-
						</button>
					</div>
				</div>

				{#if pathCommands[command[0].toUpperCase()]?.a?.length > 0}
					<hr class="my-2 mr-auto w-12" />
					<div class="flex gap-2 flex-wrap">
						{#each pathCommands[command[0].toUpperCase()].a as a, j}
							<div class="input-group input-group-divider grid-cols-[auto_1fr] w-28">
								<div class="input-group-shim">{a}</div>
								<input type="number" class="input" bind:value={parth[i][j + 1]} maxlength="1" />
							</div>
						{/each}
					</div>
				{/if}
			</div>
		{:else}
			<button on:click={() => (parth = [['M', '0', '0']])} class="mx-auto btn-icon btn-icon-sm variant-soft-primary">
				+
			</button>
		{/each}

		{#if parth.length > 0}
			<button on:click={() => (parth = [])} class="mx-auto btn btn-sm variant-soft-primary mt-4"> Clear </button>
		{/if}
	</div>
</div>

<div class="p-3 col-span-2">
	<svg viewBox="0 0 {$s.size.w} {$s.size.h}" class="w-full h-full">
		<rect x="0" y="0" width={$s.size.w} height={$s.size.h} fill="#00000010" />

		{#each Array($s.size.w) as _, x}
			{#each Array($s.size.h) as _, y}
				<rect {x} {y} width="1" height="1" fill="none" stroke="#00000010" stroke-width="0.05" />
			{/each}
		{/each}

		<path d={stringifySvgPath(parth)} fill="#FF000040" stroke="#00FF0040" />

		{#if showoff}
			<circle cx={showoff.x} cy={showoff.y} r="0.25" fill="#0000FF" />
		{/if}
	</svg>
</div>
