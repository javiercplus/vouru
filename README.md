# Vouru – Void User Repository Helper

Vouru es un script Bash para construir e instalar paquetes desde `void-packages` y el overlay comunitario **VUR**.

- Clona `void-packages`, ejecuta `binary-bootstrap`
- Busca, compila e instala paquetes
- Detecta si un paquete está en repos oficiales o solo como template
- Operaciones pesadas con spinner y log en `~/.vouru.log`

## Requisitos

- `xtools`, `git`, `curl` (los instala el instalador automático)
- `doas` o `sudo` para operaciones privilegiadas

## Instalación

```bash
# Rolling (recomendado: siempre la última versión)
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru-install-rolling | bash

# Manual
curl -sSL -o vouru https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru
chmod +x vouru && sudo mv vouru /usr/local/bin/

# Sin instalar (desde la web)
curl -sSL https://github.com/javiercplus/vouru/raw/refs/heads/main/vouru | bash -s -- search firefox
```

> El `-s --` tras `bash` es obligatorio: el `-s` lee el script desde stdin y el `--` impide que bash interprete las opciones de vouru (`-i`, `-s`, `-c`...) como si fueran suyas.

## Comandos

| Comando | Descripción |
|---------|-------------|
| `vouru add <git-url>` | Clona un repo y copia sus templates a `srcpkgs` (soporta subdirectorios y template en la raíz; omite los ya existentes) |
| `vouru search <term>` | Busca paquetes en `srcpkgs` que coincidan con `<term>` |
| `vouru install <term>` | Instala un paquete: elige entre `[repo]` (xbps-install, binario) o `[source]` (compilar con xbps-src) |
| `vouru remove <pkg>` | Elimina un paquete con `xbps-remove -y -o` (incluye huérfanos) |
| `vouru update` | Actualiza repo, `xbps-src` y el sistema |
| `vouru clean` | Borra `void-packages` y limpia caché con `xbps-remove -O` |
| `vouru set-repo [path]` | Cambia la ubicación del repo (guardado en `~/.voururc`) |

**Aliases:** `-s` (search), `-i` (install), `-r` (remove), `-c` (clean), `-a` (add)

## Ejemplos

```bash
vouru add https://codeberg.org/Neko-Void/zig-nk.git
vouru search firefox
vouru install vlc
vouru remove cmuspp-void
vouru update
vouru clean
```

## Configuración

- **Repo:** `~/.voururc` o `vouru set-repo <path>`
- **Log:** `~/.vouru.log` (revisar si clonar/bootstrap falla)

## Troubleshooting

| Problema | Solución |
|----------|----------|
| `'xtools' is not installed` | `sudo xbps-install -S xtools` |
| `Neither 'doas' nor 'sudo' found` | Instala uno de ellos |
| Compilación falla | Revisa la salida completa y las dependencias del `template` |
| `binary-bootstrap` falla | Ejecútalo manualmente en `void-packages` y revisa el error |
| Spinner colgado | `Ctrl+C`, luego revisa `~/.vouru.log` |

## License

BSD 3-Clause
