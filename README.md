# Backfilling en Azure Data Factory
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fguidofranco%2Fbackfill_data_factory%2Fmain%2Fadf_backfill_example_template.json)

## Introducción
Supongamos como escenario hipotético que debes extraer datos de la siguiente API: [Luchtmeetnet](https://api-docs.luchtmeetnet.nl/). Esta API proporciona, entre otras cosas, mediciones de la calidad del aire por medio de un endpoint `/measurements`, las cuáles se actualizan cada hora. 

Podemos desarrollar un pipeline que periódicamente obtenga los datos de la última hora, aprovechando los parámetros de la API que permiten filtrar por un rango de fechas. Ahora bien, para un análisis histórico sería necesario recuperar datos del último año, por ejemplo. Sin embargo, la API no permite filtrar por un rango de fechas tan amplio, solo admite un rango de 7 días como máximo.

Esto nos llevaría a desarrollar dos pipelines:
- uno que se ejecute periódicamente para una extracción incremental
- y otro que se ejecute por única vez para una extracción full, que vaya solicitando por ventanas de tiempo, debido a la limitación mencionada.

Ante este escenario vamos a ver como la técnica de **backfilling** nos simplifica esta implementación.

## Backfilling
El backfilling es una técnica de ingeniería de datos que nos ayuda en la recuperación de datos faltantes. En el caso de un pipeline, el backfilling nos permite recuperar datos históricos que no pudieron ser procesados en su momento. Esto es útil cuando, por ejemplo:
- se modificó la lógica de un ETL y se necesita re-procesar datos antiguos para que se ajusten a la nueva lógica.
- hubo un error en la ejecución de un proceso y se necesita recuperar los datos que no se procesaron.
- O bien, en este caso, cuando se necesita recuperar datos históricos de un origen.

## Implementación
Aplicaremos esta técnica en Azure Data Factory, por medio un `tumbling window trigger` que se encargará de ejecutar el pipeline de backfilling.

El pipeline, en si, espera una fecha y hora de inicio para poder solicitar datos de la hora anterior. Una vez obtenidos esos datos, se guardan como un archivo indicando en su nombre la fecha y hora de solicitud.

El tipo de trigger a utilizar, `tumbling window trigger`. Una de las características de este tipo de trigger es que nos permite retroceder en el tiempo y ejecutar el pipeline para ventanas de tiempo anteriores. De eso se trata el backfilling!
Podemos definir como fecha de inicio del trigger una fecha pasada como el primer día de este año y un intervalo de ejecución de 1 día. Así tendremos varias ejecuciones programadas para poder recuperar los datos de todo el año y de forma controlada ya que cada ejecución se encargará de recuperar los datos de una hora en particular sin saturar a la API.

Lo importante es que en un solo desarrollo hemos unificado la obtención de datos históricos y la extracción incremental, simplificando la implementación y mantenimiento del pipeline.
