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
    let tempoFinalizado: boolean = $state(false); 
    let intervaloTempo: ReturnType<typeof setInterval> | undefined;
    
    // Instancia de audio global para poder detenerla cuando le hagas caso
    let audioAlarma: HTMLAudioElement | null = null;

    function iniciarTemporizador(minutos?: number): void {
        detenerAlarmaSonando(); // Limpiamos cualquier alarma previa
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
                tempoFinalizado = true; // Activa el parpadeo

                // Creamos el audio en bucle con un sonido más molesto y penetrante
                audioAlarma = new Audio('https://actions.google.com/sounds/v1/alarms/digital_watch_alarm_long.ogg');
                audioAlarma.loop = true; // ¡No dejará de sonar!
                audioAlarma.play().catch(() => {
                    // Evita bloqueo de autoplay si el navegador se pone terco
                });
            }
        }, 1000);
    }

    function detenerAlarmaSonando() {
        if (audioAlarma) {
            audioAlarma.pause();
            audioAlarma.currentTime = 0;
            audioAlarma = null;
        }
        tempoFinalizado = false;
    }

    function detenerTemporizador(): void {
        tempoActivo = false;
        if (intervaloTempo) clearInterval(intervaloTempo);
        detenerAlarmaSonando();
    }

    function reiniciarTemporizador(): void {
        tempoActivo = false;
        if (intervaloTempo) clearInterval(intervaloTempo);
        detenerAlarmaSonando();
        tempoSegundos = 0;
    }

    // --- UTILIDADES ---
    function formatearTiempo(totalSegundos: number): string {
        const m = Math.floor(totalSegundos / 60).toString().padStart(2, '0');
        const s = (totalSegundos % 60).toString().padStart(2, '0');
        return `${m}:${s}`;
    }
</script>

<main class="contenedor" class:alerta-parpadeo={tempoFinalizado}>
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
                    {:else if tempoFinalizado}
                        <!-- Botón especial súper llamativo para apagar el escándalo -->
                        <button onclick={reiniciarTemporizador} class="btn-alerta-apagar">¡Hacer Caso / Apagar!</button>
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
    :global(*), :global(*::before), :global(*::after) {
        box-sizing: border-box;
    }

    :global(html), :global(body) {
        margin: 0;
        padding: 0;
        width: 100%;
        height: 100%;
        overflow: hidden; 
    }

    :global(body) {
        background-color: #050505;
        color: #ffffff;
        font-family: system-ui, -apple-system, sans-serif;
        display: flex;
        justify-content: center;
        align-items: center;
        transition: background-color 0.3s;
    }

    .contenedor {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        gap: 3rem;
        width: 100%;
        height: 100%;
        padding: 1rem;
        transition: background-color 0.3s ease;
        border-radius: 12px;
    }

    @keyframes parpadeoAlerta {
        0% { background-color: rgba(255, 0, 0, 0.6); }
        50% { background-color: rgba(0, 0, 0, 0.8); }
        100% { background-color: rgba(255, 0, 0, 0.6); }
    }

    .alerta-parpadeo {
        animation: parpadeoAlerta 0.6s infinite ease-in-out;
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
        font-size: clamp(3.5rem, 15vw, 5rem);
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
        width: 100%;
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

    .btn-alerta-apagar {
        background-color: #ff0055;
        color: #fff;
        animation: parpadeoAlerta 0.4s infinite alternate;
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
            font-size: clamp(6rem, 12vw, 10rem);
        }
        .contenedor {
            gap: 5rem;
        }
    }
</style>
