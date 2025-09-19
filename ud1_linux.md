# 📝 Prácticas RA1 – Introducción a Linux y Procesos (UD01) 
# Realiza fork, crea una carpeta UD1 donde guardes

## 🎯 Objetivo general
- Familiarizarse con Linux como entorno de ejecución.  
- Conocer cómo el sistema operativo gestiona procesos y recursos.  
- Relacionar comandos prácticos con conceptos de concurrencia, planificación y PCB.  

---

## 1. Primeros pasos: conocer el sistema
**Comandos a usar**
```bash
uname -a           # Información del kernel y SO
lsb_release -a     # Distribución (si está disponible)
hostnamectl        # Nombre del host y versión
lscpu              # Info CPU
free -h            # Memoria
df -h              # Espacio en disco
```

---

## 2. Procesos en ejecución
**Comandos a usar**
```bash
ps -ef             # Lista de procesos con detalles
top                # Monitorización en tiempo real
htop               # Versión más visual (si está instalada)
pstree             # Árbol de procesos
```

**Ejercicio**:
- Identifica el proceso padre de tu shell.  
- Localiza procesos relacionados con `systemd`.  
- Comprueba qué procesos consumen más CPU.  

---

## 3. Creación y control de procesos
**Comandos a usar**
```bash
sleep 60 &         # Crear un proceso en segundo plano
jobs               # Ver procesos de la shell
fg %1              # Traer al primer plano
kill -9 <PID>      # Finalizar un proceso
```
---

## 4. Servicios en Linux
**Comandos a usar**
```bash
systemctl --type=service     # Servicios activos
systemctl list-unit-files    # Lista de servicios instalados
```

**Ejercicio**:  
- Localiza al menos 3 servicios del sistema y explica qué función cumplen.  
- ¿Qué ocurriría si se detiene `sshd` en un servidor?  

---

## 5. Usuarios y permisos
**Comandos a usar**
```bash
whoami              # Usuario actual
id                  # UID, GID y grupos
ls -l               # Permisos de archivos
chmod, chown, chgrp # Cambiar permisos y propietarios
```

**Ejercicio**:  
1. Crea un archivo `mi_prueba.txt`.  
2. Modifica los permisos para que solo el propietario pueda leerlo.  
3. Verifica con `ls -l`.  

---

## 6. Simulación de concurrencia
**Ejercicio práctico**  
1. Abre dos terminales en Linux.  
2. En la primera, ejecuta:  
   ```bash
   yes > /dev/null
   ```  
   (consume CPU).  
3. En la segunda, ejecuta:  
   ```bash
   ls -R /
   ```  
   (consume E/S de disco).  
4. Usa `top` para observar cómo el sistema gestiona ambos procesos.  

---

## 7. Práctica con Java
Crea un programa Java que:
- Lance **2 hilos**: uno que haga cálculos intensivos, otro que escriba en un archivo.  
- Usa `Thread.sleep()` para simular espera.  
- Observa en Linux (`top`, `ps`, `htop`) cómo aparecen como subprocesos del JVM.  
