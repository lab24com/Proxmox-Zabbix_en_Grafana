# Dashboards de Grafana para Proxmox VE con Zabbix | LAKA

**Desarrollado por LAKA Soluciones Tecnológicas**

Dashboards para visualizar hipervisores y máquinas virtuales de Proxmox VE monitoreados con la plantilla oficial de Zabbix **Proxmox VE by HTTP**. Incluye una vista de hipervisores y una vista de máquinas virtuales enlazadas entre sí, con una interfaz oscura pensada para el trabajo del NOC.

Este repositorio contiene dashboards JSON; los plugins se instalan por separado desde el catálogo de Grafana.

![Hipervisores](docs/captura-hipervisores.png)

![Máquinas virtuales](docs/captura-maquinas-virtuales.png)

## Funcionalidades

**Hipervisores**

- Resumen: nodos en línea, CPU y RAM promedio, almacenamiento en advertencia o crítico y problemas activos de Zabbix con su severidad máxima.
- Una tarjeta por host Proxmox con sus nodos: estado, versión de PVE, tiempo activo y, según la plantilla, iowait y carga, o número de CPU, actualizaciones pendientes e IP.
- Gauges de CPU y RAM con zonas de 75 % y 90 %, y representación circular del disco raíz con el espacio libre.
- Almacenamiento de cada nodo ordenado del más lleno al más vacío (cinco visibles y botón para ver todos).
- Búsqueda por host o nodo y filtros Todos, Con problemas y Fuera de línea.
- Las tarjetas se reparten solas según el ancho de la ventana.

**Máquinas virtuales**

- Resumen: total, en ejecución, detenidas, advertencias, críticas y sin datos.
- Tabla con estado, nombre, ID, nodo, CPU, vCPU, RAM usada y total, sistema de archivos, disco E/S, red E/S y tiempo activo.
- Orden predeterminado por mayor consumo (las críticas primero) y ordenamiento por cualquier columna.
- Pestañas por estado con contador, vista QEMU o LXC, búsqueda por VM, ID, nodo o host y 25, 50, 100 o todas las filas por página.

**Navegación**

- Botón **Ver máquinas virtuales** en el dashboard de hipervisores y **Ver hipervisores** en el de máquinas virtuales.
- **Ver sus VMs** en cada nodo abre las máquinas virtuales de ese nodo; en la tabla, el nodo de cada VM abre su hipervisor.
- Los enlaces conservan el grupo, los hosts y el rango de tiempo.

**Otros**

- Compatible con la plantilla de Zabbix 7.0 y con la de 7.2, 7.4 y 8.0; el dashboard detecta cuál está vinculada.

## Archivos

| Archivo | Función |
|---|---|
| `dashboards/proxmox-hipervisores-es.json` | Hipervisores, nodos, almacenamiento y problemas. |
| `dashboards/proxmox-maquinas-virtuales-es.json` | Máquinas virtuales y contenedores. |
| `dashboards/*-pt-BR.json`, `dashboards/*-en.json` | Los mismos dashboards en portugués e inglés. |

## Requisitos y plugins

Configuración de referencia para estos archivos:

| Componente | Requisito |
|---|---|
| Grafana | 11 o 12, compatibles con Business Text 6.x. |
| Zabbix | 7.0, 7.2, 7.4 u 8.0 con la plantilla oficial **Proxmox VE by HTTP** vinculada a los hosts. |
| Plugin Zabbix | `alexanderzobnin-zabbix-app`, instalado y habilitado; usar una versión compatible con Grafana y Zabbix. Los JSON declaran la versión 5.0.0 como referencia. |
| Plugin Business Text | `marcusolsson-dynamictext-panel`, versión 6.x. Se utiliza en ambos dashboards. |
| Fuente de datos | Zabbix configurado en Grafana, con acceso a los hosts Proxmox y a la API. |

No se necesitan otros plugins: los gauges, las barras y la representación circular del disco están implementados dentro de Business Text.

### Plantillas admitidas

Los dashboards muestran los hosts que tengan vinculada directamente una plantilla cuyo nombre contenga **Proxmox**, como:

- `Proxmox VE by HTTP` de Zabbix 7.0 (claves `proxmox.*`)
- `Proxmox VE by HTTP` de Zabbix 7.2, 7.4 u 8.0 (claves `proxmox_ve.*`)

## 1. Instalar los plugins

### Desde la interfaz de Grafana

1. Ingrese con una cuenta administradora.
2. Abra **Administration → Plugins and data → Plugins**; el nombre del menú puede variar según la versión.
3. Busque e instale **Zabbix**. Luego abra el plugin y pulse **Enable**.
4. Busque e instale **Business Text**, en una versión 6.x compatible.

### Desde Linux

Para una instalación de Grafana administrada como servicio, puede usar estos comandos. Instalan las versiones disponibles en el catálogo; confirme su compatibilidad antes de ejecutarlos:

```bash
sudo grafana cli plugins install alexanderzobnin-zabbix-app
sudo grafana cli plugins install marcusolsson-dynamictext-panel
sudo systemctl restart grafana-server
```

Si su instalación usa el ejecutable antiguo, sustituya `grafana cli` por `grafana-cli`. Para fijar una versión, agregue su número al final del comando de instalación.

En Docker o Grafana Cloud, instale los plugins mediante el mecanismo correspondiente a su despliegue. No ejecute `systemctl` dentro de un contenedor.

## 2. Importar los dashboards

1. Descargue los dos archivos JSON del idioma que va a usar.
2. Abra **Dashboards → New → Import**.
3. Cargue `proxmox-hipervisores-es.json`.
4. En el campo **Zabbix**, seleccione la fuente de datos creada anteriormente y pulse **Import**.
5. Repita el proceso con `proxmox-maquinas-virtuales-es.json`, seleccionando la misma fuente de datos.
6. Abra el de hipervisores y deje **Grupo** y **Hosts Proxmox** en *All*, o elija los suyos.
7. Pulse **Ver máquinas virtuales** para pasar al otro dashboard.

Mantenga los UID originales para conservar los enlaces entre ambos dashboards:

| Dashboard | UID |
|---|---|
| Hipervisores | `proxmox-hipervisores-es` |
| Máquinas virtuales | `proxmox-maquinas-virtuales-es` |

En portugués e inglés el UID termina en `-pt` y `-en`.

Si ya existe un dashboard con el mismo UID, Grafana puede solicitar reemplazarlo. Para actualizar esta misma instalación, seleccione la fuente de datos y reemplace el dashboard correspondiente.

## 3. Validar la instalación

- Los hosts Proxmox deben tener ítems habilitados y datos recientes en **Monitoring → Latest data** de Zabbix.
- Compruebe que aparecen todos los nodos, con CPU, RAM, disco raíz y almacenamiento.
- En máquinas virtuales, compare el total con las VMs de Proxmox y revise nombre, ID, nodo, CPU y RAM de algunas.
- Pruebe búsqueda, filtros, pestañas, **Ver sus VMs** y los botones entre dashboards.

## Interpretación de los datos

- **Valores:** se muestra el último valor recibido por Zabbix de cada ítem, sin importar el rango de tiempo elegido. «Actualizado hace» indica cuándo se consultó la API.
- **CPU y RAM:** porcentaje usado. RAM, disco y almacenamiento: tamaños en unidades binarias (GB = 1024³ bytes).
- **Disco raíz:** porcentaje usado; el anillo y el texto muestran también el espacio libre.
- **Disco E/S:** lectura (L) y escritura (E) por segundo. **Red E/S:** recibido (↓) y enviado (↑) en bits por segundo.
- **Estado de una VM:** se toma el mayor valor entre CPU y RAM. Advertencia desde 75 %, crítico desde 90 %. *Detenida* si Proxmox informa que está apagada; *Sin datos* si está encendida pero no informa CPU ni RAM.
- **Almacenamiento:** advertencia desde 75 % y crítico desde 90 %. *Compartido* indica un almacenamiento del clúster que no pertenece a un nodo.
- **Problemas:** problemas activos de Zabbix de los hosts Proxmox, con los colores de severidad de Zabbix. *Mantenimiento* aparece cuando el host está en mantenimiento.
- **—:** sin lectura válida para esa métrica. No equivale automáticamente a un nodo o una VM caídos.

La plantilla de Zabbix 7.0 no informa vCPU ni sistemas de archivos de las VMs, por lo que esas columnas muestran «—». En LXC, la columna de sistema de archivos muestra el uso de disco del contenedor.


Los textos de los tres idiomas están en `src/i18n.json`. Las pruebas se ejecutan con `tests/run_tests.sh` (Python 3 con PyYAML y Node.js 18 o superior).

## Desarrollo y aportes

**Autor:** Willian Tola
**Marca:** LAKA Soluciones Tecnológicas
**Contacto:** +591 70806592
