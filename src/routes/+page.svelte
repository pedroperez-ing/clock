<script lang="ts">
	// Definimos los tipos estrictos para las vistas disponibles
	type Vista = 'reloj' | 'cronometro' | 'temporizador';

	// Estado global con tipos explícitos
	let vista: Vista = $state('reloj');
	let tiempoActual: Date = $state(new Date());

	// --- RELOJ ---
	$effect(() => {
		const intervalo = setInterval(() => (tiempoActual = new Date()), 1000);
		return () => clearInterval(intervalo);
	});

	// --- CRONÓMETRO ---
	let cronoSegundos: number = $state(0);
	let cronoActivo: boolean = $state(false);
	let intervaloCrono: ReturnType<typeof setInterval> | undefined;

	function toggleCrono(): void {
		cronoActivo = !cronoActivo;
		if (cronoActivo) {
			intervaloCrono = setInterval(() => cronoSegundos++, 1000);
		} else {
			if (intervaloCrono) clearInterval(intervaloCrono);
		}
	}

	function resetCrono(): void {
		cronoActivo = false;
		if (intervaloCrono) clearInterval(intervaloCrono);
		cronoSegundos = 0;
	}

	// --- TEMPORIZADOR ---
	let inputMinutos: number = $state(5);
	let tempoSegundos: number = $state(0);
	let tempoActivo: boolean = $state(false);
	let intervaloTempo: ReturnType<typeof setInterval> | undefined;

	function iniciarTemporizador(minutos?: number): void {
		if (minutos !== undefined) {
			inputMinutos = minutos;
			tempoSegundos = inputMinutos * 60;
		} 
		else if (tempoSegundos === 0) {
			tempoSegundos = inputMinutos * 60;
		}
		
		tempoActivo = true;
		if (intervaloTempo) clearInterval(intervaloTempo);
		
		intervaloTempo = setInterval(() => {
			if (tempoSegundos > 0) {
				tempoSegundos--;
			} else {
				if (intervaloTempo) clearInterval(intervaloTempo);
				tempoActivo = false;
				const audio = new Audio('https://actions.google.com/sounds/v1/alarms/beep_short.ogg');
				audio.play().catch(() => {
					// Evita excepciones flotantes si el navegador bloquea el autoplay antes de interactuar
				});
			}
		}, 1000);
	}

	function detenerTemporizador(): void {
		tempoActivo = false;
		if (intervaloTempo) clearInterval(intervaloTempo);
	}

	function reiniciarTemporizador(): void {
		tempoActivo = false;
		if (intervaloTempo) clearInterval(intervaloTempo);
		tempoSegundos = 0;
	}

	// --- UTILIDADES ---
	function formatearTiempo(totalSegundos: number): string {
		const m = Math.floor(totalSegundos / 60).toString().padStart(2, '0');
		const s = (totalSegundos % 60).toString().padStart(2, '0');
		return `${m}:${s}`;
	}
</script>

<main class="contenedor">
	<nav class="menu">
		<button class:activo={vista === 'reloj'} onclick={() => (vista = 'reloj')}>Reloj</button>
		<button class:activo={vista === 'cronometro'} onclick={() => (vista = 'cronometro')}>Cronómetro</button>
		<button class:activo={vista === 'temporizador'} onclick={() => (vista = 'temporizador')}>Temporizador</button>
	</nav>

	<div class="pantalla-principal">
		{#if vista === 'reloj'}
			<h1 class="texto-brillante">
				{tiempoActual.toLocaleTimeString('es-MX', { hour12: false })}
			</h1>
		
		{:else if vista === 'cronometro'}
			<h1 class="texto-brillante">{formatearTiempo(cronoSegundos)}</h1>
			<div class="controles">
				<button onclick={toggleCrono}>{cronoActivo ? 'Pausar' : 'Iniciar'}</button>
				<button onclick={resetCrono} class="btn-secundario">Reiniciar</button>
			</div>
		
		{:else if vista === 'temporizador'}
			<h1 class="texto-brillante">
				{formatearTiempo(tempoSegundos > 0 ? tempoSegundos : inputMinutos * 60)}
			</h1>
			
			{#if !tempoActivo && tempoSegundos === 0}
				<div class="input-grupo">
					<input type="number" bind:value={inputMinutos} min="1" class="input-tiempo" />
					<span>minutos</span>
				</div>
				<div class="controles">
					<button onclick={() => iniciarTemporizador()}>Iniciar</button>
					<button onclick={() => iniciarTemporizador(3)} class="btn-secundario">Llenado Rápido (3m)</button>
					<button onclick={() => iniciarTemporizador(5)} class="btn-secundario">Llenado Normal (5m)</button>
				</div>
			{:else}
				<div class="controles">
					{#if tempoActivo}
						<button onclick={detenerTemporizador}>Pausar</button>
					{:else}
						<button onclick={() => iniciarTemporizador()}>Reanudar</button>
					{/if}
					<button onclick={reiniciarTemporizador} class="btn-secundario">Cancelar / Reiniciar</button>
				</div>
			{/if}
		{/if}
	</div>
</main>

<style>
	:global(body) {
		margin: 0;
		background-color: #050505;
		color: #ffffff;
		font-family: system-ui, -apple-system, sans-serif;
		display: flex;
		justify-content: center;
		align-items: center;
		min-height: 100vh;
	}

	.contenedor {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 3rem;
		width: 100%;
		padding: 2rem;
	}

	.menu {
		display: flex;
		gap: 1rem;
		background: #111;
		padding: 0.5rem;
		border-radius: 50px;
		border: 1px solid #333;
	}

	.menu button {
		background: transparent;
		color: #888;
		border: none;
		padding: 0.75rem 1.5rem;
		border-radius: 40px;
		cursor: pointer;
		font-size: 1rem;
		transition: all 0.3s ease;
	}

	.menu button:hover {
		color: #fff;
	}

	.menu button.activo {
		background: #222;
		color: #fff;
		box-shadow: 0 0 10px rgba(138, 43, 226, 0.3);
	}

	.texto-brillante {
		font-size: 5rem;
		font-weight: bold;
		margin: 0;
		text-align: center;
		text-shadow: 
			0 0 10px rgba(138, 43, 226, 0.6), 
			0 0 25px rgba(138, 43, 226, 0.4), 
			0 0 50px rgba(65, 105, 225, 0.4), 
			0 0 80px rgba(65, 105, 225, 0.2);
	}

	.pantalla-principal {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 2rem;
	}

	.controles {
		display: flex;
		gap: 1rem;
		flex-wrap: wrap;
		justify-content: center;
	}

	button {
		background-color: #fff;
		color: #000;
		border: none;
		padding: 1rem 2rem;
		font-size: 1.1rem;
		font-weight: bold;
		border-radius: 8px;
		cursor: pointer;
		transition: transform 0.1s, box-shadow 0.3s;
	}

	button:active {
		transform: scale(0.95);
	}

	button:hover {
		box-shadow: 0 0 15px rgba(255, 255, 255, 0.3);
	}

	.btn-secundario {
		background-color: #222;
		color: #fff;
		border: 1px solid #444;
	}

	.btn-secundario:hover {
		box-shadow: 0 0 15px rgba(138, 43, 226, 0.4);
	}

	.input-grupo {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		font-size: 1.2rem;
	}

	.input-tiempo {
		background: #111;
		color: #fff;
		border: 1px solid #444;
		padding: 0.5rem;
		font-size: 1.5rem;
		width: 80px;
		text-align: center;
		border-radius: 8px;
	}

	@media (min-width: 768px) {
		.texto-brillante {
			font-size: 10rem;
		}
		.contenedor {
			gap: 5rem;
		}
	}
</style>