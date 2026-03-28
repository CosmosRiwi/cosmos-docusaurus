# Instalación de Docker en el VPS

---

## Pasos de Instalación

### Paso 1 — Eliminar versiones anteriores de Docker

Antes de instalar, es importante limpiar cualquier versión antigua o conflictiva que pueda existir en el sistema.

```bash
sudo apt-get remove docker docker-engine docker.io containerd runc
```

> **Nota:** Si no tienes ninguna versión instalada previamente, este comando simplemente no encontrará nada y continuará sin errores. Es seguro ejecutarlo siempre.

---

### Paso 2 — Actualizar el índice de paquetes

```bash
sudo apt-get update
```

---

### Paso 3 — Instalar dependencias necesarias

Instalamos los paquetes requeridos para que `apt` pueda usar repositorios sobre HTTPS y verificar firmas GPG.

```bash
sudo apt-get install ca-certificates curl gnupg
```

| Paquete           | Descripción                                   |
| ----------------- | --------------------------------------------- |
| `ca-certificates` | Certificados SSL para conexiones seguras      |
| `curl`            | Herramienta para descargar archivos desde URL |
| `gnupg`           | Para verificar la firma GPG del repositorio   |

---

### Paso 4 — Crear directorio para las llaves GPG

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

> Este directorio almacenará de forma segura las llaves de confianza de repositorios externos.

---

### Paso 5 — Descargar y agregar la llave GPG oficial de Docker

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```

> **¿Por qué esto?** La llave GPG garantiza que los paquetes que descargamos provienen **oficialmente de Docker Inc.** y no han sido modificados.

---

### Paso 6 — Agregar el repositorio oficial de Docker

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

> **Tip:** Este comando detecta automáticamente la arquitectura de tu servidor (`amd64`, `arm64`, etc.) y la versión de Ubuntu (`focal`, `jammy`, `noble`) para configurar el repositorio correcto.

---

### Paso 7 — Actualizar el índice con el nuevo repositorio

```bash
sudo apt-get update
```

---

### Paso 8 — Instalar Docker Engine y sus componentes

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

| Paquete                 | Descripción                                   |
| ----------------------- | --------------------------------------------- |
| `docker-ce`             | Docker Engine (Community Edition)             |
| `docker-ce-cli`         | Interfaz de línea de comandos de Docker       |
| `containerd.io`         | Runtime de contenedores                       |
| `docker-buildx-plugin`  | Plugin para builds avanzados multi-plataforma |
| `docker-compose-plugin` | Plugin para Docker Compose (`docker compose`) |

---

## Verificar la Instalación

### Comprobar que contenedores de Docker

```bash
docker --version
docker ps
```

---

## Comandos Básicos de Referencia

```bash
# Ver contenedores en ejecución
docker ps

# Ver todos los contenedores (incluyendo detenidos)
docker ps -a

# Ver imágenes descargadas
docker images

# Detener un contenedor
docker stop <nombre_o_id>

# Eliminar un contenedor
docker rm <nombre_o_id>

# Ver logs de un contenedor
docker logs <nombre_o_id>
```

---

_Documentación generada por el tren cosmos en riwi_
