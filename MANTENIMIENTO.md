# Manual de usuario de Biodont

- **`Manual-Biodont.html`** — el manual. Se abre con doble clic en cualquier navegador.
  No necesita internet ni servidor.
- **`capturas/`** — las imágenes que usa. **Viaja con el HTML**: si se copia el manual a otro
  equipo, hay que copiar la carpeta entera, no solo el archivo.

## Cómo se entrega al consultorio

Copiar la carpeta `manual/` completa. Para dejarlo en PDF: abrir el HTML y usar
*Imprimir → Guardar como PDF*. La hoja de estilos ya oculta el índice lateral al imprimir.

## Cómo se hicieron las capturas

Con el sistema corriendo sobre una **base de datos de ejemplo**, nunca sobre datos reales.
Los pacientes que aparecen (María Rodríguez, Andrea Salazar…) son ficticios y vienen del seed
de demostración.

```bash
# 1. Base limpia, aparte de la del consultorio
export DATABASE_URL="file:/ruta/a/demo.db"
npx prisma migrate deploy
node prisma/seed.js                  # usuario administrador
node prisma/seed-procedimientos.js   # catálogo CUPS — ANTES que los permisos
node prisma/seed-permisos.js
node prisma/seed-demo.js             # 55 pacientes ficticios y su actividad

# 2. Backend contra esa base
PORT=3055 DATABASE_URL="file:/ruta/a/demo.db" node src/server.js

# 3. Front apuntando ahí (environment.ts → apiUrl) y capturas a 1440x900
```

⚠️ **El seed de demostración no siembra ni alergias estructuradas, ni PIN de firma, ni
presupuestos, ni cotizaciones.** Para las capturas se crearon a mano por la vía normal de la
aplicación. Sin eso no se puede demostrar la alerta clínica ni la firma.

## Los diagramas NO se editan aquí

Los seis diagramas del manual son SVG escritos dentro del HTML —sin librerías ni imágenes
externas, para que siga abriéndose con doble clic y sin internet— pero **se generan**, no se
dibujan a mano: el generador vive en `Biodont/manual/_gen/` dentro del repositorio del
sistema. Editar el SVG aquí se pierde en la siguiente generación.

El motivo de generarlos es el mismo que el de medir la tabla de roles: un diagrama hecho a
mano se desincroniza del sistema en el primer cambio de rótulo, y nadie lo nota porque un
dibujo no falla. Validar el manual con cada actualización de producción tiene que poder ser
correr un comando, no releer seis dibujos.

## Al actualizar el manual

1. **Vuelva a medir lo que afirme.** La tabla «Qué puede hacer cada rol» se construyó pidiendo
   cada ruta con el token de los cuatro roles y anotando el código de respuesta, no leyendo la
   documentación. Dos celdas estaban mal en el primer borrador (el auxiliar **no** entra a
   presupuestos ni a cotizaciones) y solo lo delató la medición.
2. **Rehaga la captura** de la pantalla que cambie. Las imágenes envejecen antes que el texto.
3. **Compruebe que no quedan enlaces rotos**:

```bash
python3 - <<'PY'
import re, os
h = open('Manual-Biodont.html', encoding='utf-8').read()
imgs = re.findall(r'<img src="([^"]+)"', h)
print('imágenes rotas:', [s for s in imgs if not os.path.exists(s)])
ids = set(re.findall(r'id="([^"]+)"', h))
print('anclas sin destino:', sorted(set(re.findall(r'href="#([^"]+)"', h)) - ids))
PY
```

4. **Si prueba el manual en un servidor local, fuerce la recarga.** `python3 -m http.server` no
   manda cabeceras anti-caché y el navegador sirve la versión anterior: parece que su cambio no
   se guardó cuando sí lo hizo.

## Relación con los manuales anteriores

En el repositorio del sistema existen dos manuales anteriores (`MANUAL.md` y
`MANUAL-USUARIO-COMPLETO.md`) que **no se han tocado**. Este los reemplaza para el uso del
consultorio: aquellos están organizados por módulos del software, no llevan capturas, y no
cubren cotizaciones, actuaciones, rechazo de tratamiento, revocatoria, reprogramación de citas
ni alertas clínicas.

## De dónde sale este repositorio

La copia de trabajo vive en `Biodont/manual/` dentro del repositorio del sistema. Este
repositorio es **la entrega**: al actualizar el manual hay que copiar aquí el HTML y las
capturas, y volver a publicar.
