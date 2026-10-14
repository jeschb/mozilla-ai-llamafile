# TUTORIAL PRÁCTICO: Instalación y uso de llamafile

**Autor:** Ing. Jesús A. Chávez Becerra

[llamafile](https://github.com/mozilla-ai/llamafile) es un proyecto de
[Mozilla.ai](https://www.mozilla.ai/open-tools/llamafile) que empaqueta un modelo
de lenguaje (LLM) junto con el motor de inferencia [llama.cpp](https://github.com/ggml-org/llama.cpp)
en **un único archivo ejecutable**.

Su gran ventaja: **no requiere instalación**. Solo descargas el archivo, le das
permisos de ejecución y lo lanzas. Funciona en la gran mayoría de sistemas
operativos y arquitecturas, sin dependencias externas gracias a
[Cosmopolitan Libc](https://github.com/jart/cosmopolitan)).

**Versión usada en este tutorial:** `llamafile 0.10.6`

**Validado en:** Ubuntu 26.04.1 LTS (x86_64, kernel 7.0.0-38-generic) con el
archivo `Qwen3.5-0.8B-Q8_0.llamafile` existente en este equipo.

---

## 1. Entorno de trabajo

Estos son los recursos detectados y validados en esta máquina:

| Recurso | Valor detectado |
|---|---|
| Sistema operativo | Ubuntu 26.04.1 LTS · x86_64 · kernel 7.0.0-38-generic |
| CPU | 16 vCPU (QEMU Virtual CPU, virtualizado con KVM) |
| Memoria RAM | 17 GiB total · ~11–14 GiB libres/disponibles |
| GPU | Ninguna (Red Hat Virtio 1.0 GPU, QXL virtual) → **inferencia 100 % en CPU** |
| Disco libre | ~79 GB en `/` |
| Extensiones CPU | **Solo `sse4_2`**. Sin `avx`, `avx2`, `fma`, `f16c` |

> **Aviso importante sobre rendimiento**
>
> Al estar virtualizado, esta CPU **no expone AVX2**. Llamafile funciona perfectamente
> sin AVX, pero **la inferencia será notablemente más lenta** que en una CPU física
> moderna con AVX2. Por este motivo, en este tutorial se recomienda usar el modelo
> **más ligero** (`Qwen3.5-0.8B-Q8_0`).
>
> Para comprobar tus extensiones CPU desde terminal:
>
> ```sh
> grep -m1 '^flags' /proc/cpuinfo | tr ' ' '\n' | grep -E 'avx|fma|f16c' | sort -u
> ```
>
> Si no muestra nada, significa que **no hay AVX disponible**.

**Herramientas ya presentes:** `curl`, `wget`, `git`. No es necesario compilar nada.

---

## 2. Modelos recomendados para este entorno

Todos los modelos listados pertenecen a la familia **Qwen3.5** (licencia Apache 2.0),
son **multimodales** (admiten imágenes con el comando `/upload` desde el chat) y
están disponibles en [`mozilla-ai/llamafile_0.10`](https://huggingface.co/mozilla-ai/llamafile_0.10).

| Modelo | Tamaño | RAM aprox. | Velocidad (sin AVX2) | Calidad | Uso recomendado |
|---|---|---|---|---|---|
| **`Qwen3.5-0.8B-Q8_0.llamafile`** | 1,23 GB | ~1,5–2 GB | **Más rápido** | Suficiente para uso general | **Recomendado para este entorno** |
| **`Qwen3.5-2B-Q8_0.llamafile`** | 2,89 GB | ~3–4 GB | Rápido | Buena | Buen equilibrio si el 0,8B se queda corto |
| **`Qwen3.5-4B-Q5_K_S.llamafile`** | 3,91 GB | ~4–5 GB | Medio-lento | Muy buena | Más preciso, pero más lento |
| **`Qwen3.5-9B-Q5_K_S.llamafile`** | 7,49 GB | ~7–8 GB | **Más lento** | Óptima | Solo si necesitas máxima calidad y tienes paciencia |

**Mi recomendación personal para esta máquina:** empezar con **Qwen3.5-0.8B-Q8_0**.
Si con el tiempo notas que las respuestas se quedan cortas, sube al **2B**.

---

## 3. Método A – Archivo `.llamafile` autocontenido (Recomendado)

Este es el método más práctico. El propio ejecutable ya lleva incrustado tanto
el motor `llamafile/llama.cpp` como el modelo GGUF.

Nos situamos en la carpeta donde vamos a trabajar:

```sh
cd /home/jesus/Documentos/llamafile
```

### Paso 1. Descargar el modelo (si no lo tienes)

Ya tienes `Qwen3.5-0.8B-Q8_0.llamafile` descargado. Si quisieras otro modelo,
sustituye la URL por el correspondiente:

```sh
curl -LO https://huggingface.co/mozilla-ai/llamafile_0.10/resolve/main/Qwen3.5-0.8B-Q8_0.llamafile
```

### Paso 2. Dar permisos de ejecución

Solo la primera vez:

```sh
chmod +x Qwen3.5-0.8B-Q8_0.llamafile
```

### Paso 3. Lanzar llamafile

```sh
./Qwen3.5-0.8B-Q8_0.llamafile
```

Con este comando se inicia en **modo combinado**: abre un **chat interactivo en
terminal** y, al mismo tiempo, levanta un **servidor web + API** en
[http://localhost:8080](http://localhost:8080).

Puedes hablar directamente por terminal, o abrir ese enlace en tu navegador web.

---

## 4. Método B – Binario `llamafile` + modelo GGUF externo

Útil si prefieres mantener el motor separado del archivo de pesos, o reutilizar
distintos modelos `.gguf`.

```sh
cd /home/jesus/Documentos/llamafile

# 1) Descargar el motor llamafile 0.10.6
curl -LO https://github.com/mozilla-ai/llamafile/releases/download/0.10.6/llamafile-0.10.6
chmod +x llamafile-0.10.6

# 2) Descargar un modelo GGUF (ejemplo)
curl -LO https://huggingface.co/mozilla-ai/llamafile_0.10/resolve/main/Qwen3.5-2B-Q8_0.llamafile

# 3) Ejecutar pasando el modelo con -m
./llamafile-0.10.6 -m Qwen3.5-2B-Q8_0.llamafile
```

> **Nota:** El binario `llamafile-0.10.6` incluye librerías para GPU, pero al
> **no existir GPU** en esta máquina se ejecutará íntegramente en CPU. Si quieres
> algo aún más ligero, existe `llamafile-0.10.6-thin` en la misma release.

---

## 5. Modos de lanzamiento

Tanto con un `.llamafile` autocontenido, como con `llamafile -m modelo.gguf`,
funcionan los mismos argumentos.

### A) Modo combinado (por defecto) – Recomendado para uso local

**Chat en terminal + Servidor Web + API OpenAI-compatible al mismo tiempo.**

```sh
./Qwen3.5-0.8B-Q8_0.llamafile
```

Accesible en: [http://localhost:8080](http://localhost:8080)

### B) Modo servidor (`--server`) – Solo API + Web UI

**NO muestra chat en terminal. Únicamente levanta el servidor HTTP.**

Ideal si vas a usarlo con agentes, scripts, curl, OpenWebUI, Continue, etc.

```sh
./Qwen3.5-0.8B-Q8_0.llamafile --server
```

Accesible en: [http://localhost:8080](http://localhost:8080)

### C) Modo chat (`--chat`) – Solo terminal

**Únicamente interfaz interactiva por terminal. NO levanta servidor web.**

```sh
./Qwen3.5-0.8B-Q8_0.llamafile --chat
```

Comandos útiles dentro del chat: `/help`, `/upload ruta/imagen.jpg`, `Ctrl+C` para salir.

### D) Modo CLI (`--cli`) – Una sola pregunta y salir

Perfecto para usarlo desde scripts o responder rápido sin entrar al chat.

```sh
./Qwen3.5-0.8B-Q8_0.llamafile --cli -p "Escribe un haiku sobre llamas"
```

> **Dato clave:** Con **A) sin `--server`** tienes **terminal + web/API**. Con
> **B) con `--server`** tienes **solo web/API**. El servidor HTTP se levanta en
> ambos casos. La única diferencia es que con `--server` **se omite el chat en
> la terminal**.

---

## 6. Acceso desde otro equipo de la misma red (LAN)

Por defecto, llamafile escucha **únicamente** en `127.0.0.1` (localhost). Esto
significa que **otros ordenadores de tu red WiFi/LAN NO podrán conectarse**.

Para permitir conexiones desde otros equipos, hay que forzarlo a escuchar en
**todas las interfaces** con `--host 0.0.0.0`.

### 6.1 Averiguar la IP local de este equipo

Desde el equipo que va a ejecutar llamafile:

```sh
hostname -I
# o
ip -4 addr show | grep -oP 'inet \K[\d.]+' | grep -v '^127\.'
```

Obtendrás algo como `192.168.1.42`. Llamaremos a esa IP `<IP_LLAMAFILE>`.

### 6.2 Lanzar escuchando en toda la red

Elige uno de estos dos casos según lo que busques:

**Opción 1: Quiero chat en terminal + acceso web/API desde red**

```sh
./Qwen3.5-0.8B-Q8_0.llamafile --host 0.0.0.0 --port 8080
```

**Opción 2: Quiero SOLO servidor (sin chat en terminal) + acceso web/API desde red**

```sh
./Qwen3.5-0.8B-Q8_0.llamafile --server --host 0.0.0.0 --port 8080
```

> **Importante:** Tanto en modo combinado como en `--server`, **ambos levantan
> el servidor HTTP**. Si vas a exponerlo a la red local, **siempre añade
> `--host 0.0.0.0`**. Sin ese flag, solo funcionará desde `localhost`.

### 6.3 Permitir el puerto en el cortafuegos (si aplica)

Si tienes `ufw` activo:

```sh
sudo ufw status
sudo ufw allow 8080/tcp
sudo ufw reload
```

Si usas `iptables` directamente:

```sh
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
```

**Si `ufw status` muestra `inactive`, puedes saltarte este paso.**

### 6.4 Conectarse desde otro equipo

Desde cualquier otro equipo conectado a la **misma red local**, abre el navegador:

```text
http://<IP_LLAMAFILE>:8080/
```

Ejemplo: `http://192.168.1.42:8080/`

Verás la interfaz web de llamafile lista para chatear.

También puedes usar la **API OpenAI-compatible** desde ese otro equipo:

```sh
curl http://<IP_LLAMAFILE>:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer no-key" \
  -d '{
    "model": "LLaMA_CPP",
    "messages": [{"role": "user", "content": "Hola desde otro equipo de la red"}]
  }'
```

### 6.5 Comprobar conectividad

Desde el **otro equipo**:

```sh
curl -I http://<IP_LLAMAFILE>:8080/
# Debe devolver: HTTP/1.1 200 OK

# Alternativa con netcat
nc -vz <IP_LLAMAFILE> 8080
# Debe decir: succeeded! o Connection succeeded
```

**Si falla desde otro equipo pero sí funciona desde `localhost`:**

1. ¿Lanzaste con `--host 0.0.0.0`? (paso 6.2)
2. ¿Permite el cortafuegos el puerto 8080? (paso 6.3)
3. ¿Ambos equipos están en la misma red y usas la IP correcta?
4. ¿Algún router/AP bloquea comunicación entre dispositivos? (poco habitual en redes domésticas)

> **Seguridad:** Con `--host 0.0.0.0` **cualquier equipo de tu red local** podrá
> acceder al modelo y a su API. Por defecto **NO hay autenticación**.
>
> Si vas a usarlo en una red con otros usuarios, añade una clave:
>
> ```sh
> ./Qwen3.5-0.8B-Q8_0.llamafile --server --host 0.0.0.0 --port 8080 --api-key MI_CLAVE_SECRETA
> ```
>
> Y luego usa: `-H "Authorization: Bearer MI_CLAVE_SECRETA"`

---

## 7. Ajustes óptimos para este entorno (16 vCPU · 17 GiB · sin GPU)

Teniendo en cuenta que **no hay AVX2** y **no hay GPU**, lo ideal es dejar
margen de CPU y no forzar contexto demasiado grande.

Mi configuración práctica para `Qwen3.5-0.8B-Q8_0`:

```sh
./Qwen3.5-0.8B-Q8_0.llamafile \
  -t 8 \        # Número de hilos CPU (deja 8 libres de 16)
  -c 4096 \     # Tamaño de contexto (suficiente para la mayoría de chats)
  -ngl 0 \      # Sin capas en GPU (obligatorio aquí)
  --no-mmap     # Evita mmap si alguna vez notas rarezas de memoria (opcional)
```

### Explicación de parámetros

| Flag | Significado | Recomendado aquí |
|---|---|---|
| `-t, --threads` | Hilos de CPU usados para generación | **8** (equilibrado entre velocidad y dejar CPU libre) |
| `-c, --ctx-size` | Tokens máximos en contexto | **4096**. Puedes subir a 6144 si sobra RAM, o bajar a 2048 si notas presión |
| `-ngl, --n-gpu-layers` | Capas a enviar a GPU | **0** (sin GPU) |
| `--no-mmap` | Desactiva memory-mapping | **Opcional**. Normalmente no hace falta |
| `-b, --batch-size` | Lote lógico | Por defecto va bien. Suele 2048 |
| `-ub, --ubatch-size` | Lote físico | Por defecto 512 |

**Trucos rápidos:**

- Si quieres **máxima fluidez** y solo usas tú el equipo → `-t 10` o `-t 12`
- Si vas a seguir usando el PC mientras corre → `-t 6`–`8`
- Si vas a exponer a red y recibir varias peticiones → mejor mantener `-t 8`

Ver todas las opciones: `./Qwen3.5-0.8B-Q8_0.llamafile --help`

---

## 8. Verificación rápida (API OpenAI-compatible)

Con llamafile corriendo (modo combinado o `--server`), abre otra terminal y prueba:

```sh
curl http://localhost:8080/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer no-key" \
  -d '{
    "model": "LLaMA_CPP",
    "messages": [
      {"role": "user", "content": "Dime en una frase qué es llamafile"}
    ],
    "max_tokens": 40
  }'
```

**Respuesta esperada:** Un JSON con `choices[0].message.content` conteniendo la
respuesta del modelo.

También puedes comprobar los modelos disponibles:

```sh
curl http://localhost:8080/v1/models
```

Y, por supuesto, abrir [http://localhost:8080](http://localhost:8080) para usar
la **interfaz web** visualmente.

---

## 9. Solución de problemas

| Problema | Solución |
|---|---|
| **Va muy lento** | Totalmente normal sin AVX2. Usa `Qwen3.5-0.8B-Q8_0`, baja hilos (`-t 6–8`) o sube un poco si solo lo usas tú (`-t 10–12`). |
| **No arranca / "Permission denied"** | Falta dar permisos: `chmod +x Qwen3.5-0.8B-Q8_0.llamafile` |
| **Se queda sin memoria** | Baja contexto: `-c 2048`. Evita usar 4B/9B si tienes muchas apps abiertas. Con 0,8B rara vez pasa. |
| **Puerto 8080 ocupado** | Cambia puerto: `--port 8081` (y usa ese puerto en navegador/curl) |
| **No accedo desde otro equipo** | Revisa: `--host 0.0.0.0` (6.2), cortafuegos (6.3), misma red, IP correcta. Prueba `nc -vz <IP> 8080` desde el otro equipo. |
| **Quiere arrancar y se cierra** | Suele ser por ruta/modelo mal indicado (Método B). Comprueba que el `.gguf` existe y se pasa con `-m`. |
| **Demasiado consumo de CPU** | Baja `-t` a 6–8 para dejar hueco al resto del sistema. |

**Parar llamafile:** Pulsa `Ctrl+C` en la misma terminal donde lo lanzaste.

**Ver ayuda completa:** `./Qwen3.5-0.8B-Q8_0.llamafile --help`

---

## 10. Recursos oficiales

- **Repositorio GitHub:** [github.com/mozilla-ai/llamafile](https://github.com/mozilla-ai/llamafile)
- **Documentación oficial:** [docs.mozilla.ai/llamafile](https://docs.mozilla.ai/llamafile)
- **Modelos pre-construidos:** [huggingface.co/mozilla-ai/llamafile_0.10](https://huggingface.co/mozilla-ai/llamafile_0.10)
- **Releases llamafile:** [github.com/mozilla-ai/llamafile/releases](https://github.com/mozilla-ai/llamafile/releases)
- **llama.cpp:** [github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)

---

**Autor y recopilador:** Ing. Jesús A. Chávez Becerra  
**Fecha:** Octubre 2025  
**Entorno validado:** Ubuntu 26.04.1 LTS · llamafile 0.10.6 · Qwen3.5-0.8B-Q8_0.llamafile

**Nota personal:** Este manual es fruto de mi propia recopilación, pruebas y validación con llamafile en este entorno.
