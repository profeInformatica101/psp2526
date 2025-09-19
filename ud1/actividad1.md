# Programación de servicios y procesos en Java (ProcessBuilder)

Este documento amplía tus apuntes en **.md** con código listo para usar y las actividades **1.2, 1.3, 1.4 y 1.5**. Está pensado para Linux/macOS (bash) y Windows (PowerShell/CMD), indicando alternativas cuando proceda.

---

## 0) Requisitos y cómo compilar

```bash
# Compilar todos los .java del directorio actual
javac *.java

# Ejecutar (ejemplos Linux/macOS)
java LanzaProceso ls /etc
java ComprobarProceso ./bucle.sh
java EjecutaComandoTimeout find / -name "*.sh"

# En Windows (CMD)
java LanzaProceso cmd /c dir C:\\Windows
java EjecutaComandoTimeout cmd /c dir /s C:\\Windows
```

> Consejo: si quieres que funcione igual en cualquier S.O., usa el envoltorio `Command.os()` mostrado más abajo.

---

## 1) Programa base: `LanzaProceso.java` (Actividad 1.2)

```java
import java.io.IOException;
import java.util.Arrays;

public class LanzaProceso {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.err.println("Debe indicarse comando a ejecutar.");
            System.exit(1);
        }

        ProcessBuilder pb = new ProcessBuilder(args);
        try {
            Process p = pb.start();
            int codRet = p.waitFor();
            System.out.println("La ejecución de " + Arrays.toString(args)
                    + " devuelve " + codRet
                    + " -> " + (codRet == 0 ? "Ejecución correcta" : "ERROR"));
        } catch (IOException e) {
            System.err.println("Error durante ejecución del proceso");
            e.printStackTrace();
            System.exit(2);
        } catch (InterruptedException e) {
            System.err.println("Proceso interrumpido");
            Thread.currentThread().interrupt();
            System.exit(3);
        }
    }
}
```

### Casos a probar (1.2)

* **a)** Proceso sin errores: `java LanzaProceso ls /etc` (o en Windows `cmd /c dir C:\\Windows`).
* **b)** Programa que no existe: `java LanzaProceso programa_que_no_existe` → `IOException`.
* **c)** Proceso con código de error: `java LanzaProceso ls /et` → `exit != 0`.

---

## 2) Comprobar si el proceso sigue vivo (Actividad 1.3)

### 2.1) Script de prueba: `bucle.sh` (≈10 s)

```bash
#!/bin/bash
i=0
while [ $i -lt 10 ]; do
  echo $i
  i=$((i + 1))
  sleep 1
done
```

```bash
chmod +x bucle.sh
```

### 2.2) `ComprobarProceso.java`

```java
import java.io.IOException;
import java.util.Arrays;

public class ComprobarProceso {
    public static void main(String[] args) {
        if (args.length == 0) {
            System.err.println("Debe indicarse comando a ejecutar.");
            System.exit(1);
        }

        ProcessBuilder pb = new ProcessBuilder(args);
        try {
            Process p = pb.start();
            while (p.isAlive()) {
                System.out.println("El proceso " + Arrays.toString(args) + " sigue en ejecución...");
                try { Thread.sleep(3000); } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
            System.out.println("Ha terminado con código: " + p.exitValue());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 3) Tiempo máximo de ejecución (timeout) y fusión de salidas

### 3.1) `EjecutaComandoTimeout.java`

```java
import java.io.IOException;
import java.util.Arrays;
import java.util.concurrent.TimeUnit;

public class EjecutaComandoTimeout {
    // milisegundos por defecto (se puede sobreescribir con un argumento opcional)
    public static int MAX_TIEMPO_MS = 5000; // 5 s por defecto

    public static void main(String[] args) {
        if (args.length == 0) {
            System.err.println("Uso: java EjecutaComandoTimeout <comando...> [--timeoutMs=N]");
            System.exit(1);
        }

        // Permite pasar --timeoutMs=NNNN como último argumento
        String[] comando = args;
        if (args[args.length-1].startsWith("--timeoutMs=")) {
            String v = args[args.length-1].substring("--timeoutMs=".length());
            MAX_TIEMPO_MS = Integer.parseInt(v);
            comando = Arrays.copyOf(args, args.length-1);
        }

        ProcessBuilder pb = new ProcessBuilder(comando);
        pb.redirectErrorStream(true); // mezcla stdout y stderr
        pb.inheritIO();               // usa la consola del padre

        System.out.printf("Se ejecuta comando: %s\n", Arrays.toString(comando));
        try {
            Process p = pb.start();
            boolean ok = p.waitFor(MAX_TIEMPO_MS, TimeUnit.MILLISECONDS);
            if (!ok) {
                System.out.printf("AVISO: No ha terminado en %d ms; se destruye el proceso...\n", MAX_TIEMPO_MS);
                p.destroy(); // o destroyForcibly() si no responde
                if (p.isAlive()) p.destroyForcibly();
            }
            if (!p.isAlive()) {
                System.out.println("Código de salida: " + p.exitValue());
            }
        } catch (IOException e) {
            System.err.println("Error durante ejecución. Información detallada:");
            e.printStackTrace();
            System.exit(2);
        } catch (InterruptedException ex) {
            System.err.println("Proceso interrumpido");
            Thread.currentThread().interrupt();
            System.exit(3);
        }
    }
}
```

**Ejemplo (Linux/macOS):**

```bash
java EjecutaComandoTimeout find / -name "*.sh" --timeoutMs=500
```

**Ejemplo (Windows):**

```bash
java EjecutaComandoTimeout cmd /c dir /s C:\\Windows --timeoutMs=500
```

---

## 4) Utilidad cross‑platform (opcional)

Para no recordar diferencias entre Windows y Unix, usa este envoltorio:

```java
final class Command {
    static String[] os(String... cmdUnix) {
        String os = System.getProperty("os.name").toLowerCase();
        if (os.contains("win")) {
            // Une los argumentos en una sola cadena para cmd.exe
            String joined = String.join(" ", cmdUnix);
            return new String[] {"cmd", "/c", joined};
        }
        return cmdUnix; // Linux/macOS
    }
}
```

Ejemplo:

```java
ProcessBuilder pb = new ProcessBuilder(Command.os("ls", "-la"));
```

---

## 5) Actividad 1.4 – Medir tiempo de ejecución y repetir en distintos directorios

**Objetivo:** crear un programa que ejecute un comando, **mida el tiempo real de ejecución** y permita **probarlo en varios directorios**. Debe mostrar código de salida y tiempo total. Opcionalmente, aceptar un `--timeoutMs`.

### 5.1) `MideTiempoProceso.java`

```java
import java.io.File;
import java.io.IOException;
import java.util.Arrays;

public class MideTiempoProceso {
    public static void main(String[] args) throws Exception {
        if (args.length < 2) {
            System.err.println("Uso: java MideTiempoProceso <dir1[:dir2[:...]]> <comando...>");
            System.err.println("Ej:   java MideTiempoProceso /etc:/var ls -la");
            System.exit(1);
        }
        String[] dirs = args[0].split(":");
        String[] comando = Arrays.copyOfRange(args, 1, args.length);

        for (String d : dirs) {
            File wd = new File(d);
            System.out.printf("\n== Ejecutando en '%s' -> %s ==\n", wd.getAbsolutePath(), Arrays.toString(comando));
            ProcessBuilder pb = new ProcessBuilder(comando);
            pb.directory(wd);
            pb.redirectErrorStream(true);
            long t0 = System.nanoTime();
            try {
                Process p = pb.start();
                int code = p.waitFor();
                long t1 = System.nanoTime();
                double ms = (t1 - t0) / 1_000_000.0;
                System.out.printf("Código: %d | Tiempo: %.2f ms\n", code, ms);
            } catch (IOException | InterruptedException e) {
                System.err.println("Fallo en " + d + ": " + e.getMessage());
                if (e instanceof InterruptedException) Thread.currentThread().interrupt();
            }
        }
    }
}
```

**Ejemplos:**

```bash
# Linux/macOS
java MideTiempoProceso /etc:/var ls -la

# Windows (dos carpetas)
java MideTiempoProceso C:\\Windows;C:\\  cmd /c dir  # (si usas ; en vez de :, adapta el split)
```

> Nota: si prefieres Windows con `;` como separador, cambia `split(":")` por `split("[:;]")`.

**Qué comentar en la memoria:** diferencias de tiempo según directorio, efecto de E/S y nº de ficheros.

---

## 6) Actividad 1.5 – Progreso y lectura en streaming

**Objetivo:** mostrar avances de ejecución **mientras el proceso está corriendo**. Leeremos su **salida estándar en streaming** y mostraremos un **spinner/contador**; al terminar, se imprime el resumen con tiempo y código de salida.

### 6.1) `EjecutaConProgreso.java`

```java
import java.io.*;
import java.time.Duration;
import java.time.Instant;
import java.util.concurrent.atomic.AtomicBoolean;

public class EjecutaConProgreso {
    public static void main(String[] args) throws Exception {
        if (args.length == 0) {
            System.err.println("Uso: java EjecutaConProgreso <comando...>");
            System.exit(1);
        }

        ProcessBuilder pb = new ProcessBuilder(args);
        pb.redirectErrorStream(true); // mezcla stdout+stderr para leer en orden

        Instant t0 = Instant.now();
        Process p = pb.start();

        // Hilo que lee la salida en tiempo real
        Thread lector = new Thread(() -> {
            try (BufferedReader br = new BufferedReader(new InputStreamReader(p.getInputStream()))) {
                String line;
                while ((line = br.readLine()) != null) {
                    System.out.println(line); // Mostrar línea del proceso
                }
            } catch (IOException e) {
                System.err.println("[lector] " + e.getMessage());
            }
        });
        lector.setDaemon(true);
        lector.start();

        // Spinner sencillo mientras el proceso está vivo
        String frames = "|/-\\"; int i = 0;
        while (p.isAlive()) {
            System.out.print("\rTrabajando " + frames.charAt(i++ % frames.length()));
            try { Thread.sleep(200); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
        }
        System.out.print("\r");

        int code = p.waitFor();
        Duration dt = Duration.between(t0, Instant.now());
        System.out.printf("Finalizado. Código=%d | Tiempo=%d ms\n", code, dt.toMillis());
    }
}
```

**Pruebas sugeridas:**

* Linux/macOS: `java EjecutaConProgreso sh -c "for i in $(seq 1 10); do echo $i; sleep 1; done"`
* Windows: `java EjecutaConProgreso powershell -Command "1..10 | ForEach-Object { $_; Start-Sleep -s 1 }"`

> Variante: cuenta cuántas líneas imprime el comando y muéstralo como *items procesados*.

---

## 7) Retos y preguntas de reflexión

* ¿Qué diferencia hay entre `inheritIO()` y leer con `getInputStream()`?
* ¿Cuándo usar `redirectErrorStream(true)`?
* ¿En qué casos conviene `destroyForcibly()`?
* Prueba a lanzar procesos que generen mucha salida (p. ej., `find /`) **sin** leer la salida: ¿qué ocurre con el buffer?

---

## 8) Checklist de entrega

* [ ] Código compila sin warnings.
* [ ] README con comandos de prueba en tu S.O.
* [ ] Capturas o logs de ejecución (1.2, 1.3, 1.4, 1.5).
* [ ] Comentario sobre tiempos y gestión de E/S.
* [ ] Explicación de decisiones de diseño (timeout, progreso, manejo de errores).

---

> **Licencia sugerida:** MIT. Añade un `Makefile` o scripts `.sh`/`.ps1` para automatizar las pruebas si lo deseas.
