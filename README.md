# Proyecto de Limpieza de Datos - Estadísticas de Juzgado

## Introducción
Este repositorio contiene el proyecto de limpieza de datos que hice usando un Excel del lugar en el cual trabajo actualmente. Incluye los archivos procesados, un archivo de Power BI para visualización, y documentación detallada del proceso realizado. Este Excel fue usado por, como mínimo, 5 personas, por lo cual contaba con inconsistencias, errores, abreviaciones mal hechas, cosas mal ingresadas, etc.

## Estructura del Proyecto

El proyecto está organizado de la siguiente manera:

```
limpieza-datos-proyecto/
├── README.md              # Documentación principal del proyecto
├── LICENSE                # Documentación respecto a la licencia del proyecto
├── data/
│   ├── original/          # Datos originales (sin información sensible)
│   │   ├── Dataset Limpio.xlsx
│   │   └── Esquema de registro original.xlsx
│   ├── procesada/         # Datos procesados y limpios
│       └── Dataset juzgado.csv
└── analisis/
    ├── Estadistica ingresos 2020-2024.pdf # Gráficos de PowerBI para una visualización general
    └── Estadistica ingresos 2020-2024.pbix # Archivo de Power BI del dashboard

```

### Descripción de Carpetas
- **data/original/**: Contiene los datos originales o semi procesados antes de la limpieza final.
- **data/procesada/**: Contiene los datasets procesados y listos para análisis.
- **analisis/**: Contiene el archivo Power BI y otros análisis derivados.

### Esquema de Registro Original
- Se subió este archivo para que se pueda ver como era el archivo xlsx original en cuanto a columnas y campos. Por cuestiones de privacidad no puedo subir el archivo original.

## Proceso de Limpieza de Datos

### 1. Revisión Inicial
- Identificación de duplicados, datos faltantes, errores de ingreso y formatos inconsistentes. En este caso no se encontraron duplicados.

### 2. Eliminación de Nombres Propios y Extracción de Delito en Crudo
En el Excel original, se encontraban cargados nombre y apellidos de personas imputadas, por lo que se procedió a eliminar gradualmente esos datos. No se podía eliminar la columna directamente porque la misma tenía el formato "nombre s/ delito", siendo este uno de los datos a analizar.

- **Fórmulas utilizadas**:
  - Para convertir el nombre en iniciales, teniendo en cuenta que el separador es "s/":
    ```excel
    =IF( G4="", "Sin nombre s/ Sin delito", IFERROR( CONCATENATE( IF( OR(ISERROR(FIND(", ",G4)), ISERROR(FIND(" ",G4))), "", LEFT(G4,1)&". " ), IF( AND(NOT(ISERROR(FIND(", ",G4))), NOT(ISERROR(FIND(" s/ ",G4)))), LEFT(MID(G4,FIND(", ",G4)+2,FIND(" s/ ",G4)-FIND(", ",G4)-2),1)&". ", IF( AND(NOT(ISERROR(FIND(" ",G4))), NOT(ISERROR(FIND(" s/ ",G4)))), LEFT(MID(G4,FIND(" ",G4)+1,FIND(" s/ ",G4)-FIND(" ",G4)-1),1)&". ", "" ) ), IFERROR( MID(G4,FIND(" s/ ",G4)+1,LEN(G4)-FIND(" s/ ",G4)), "Sin delito" ) ), "Sin nombre s/ Sin delito" ) )
    ```
  - Para separar el delito de la columna:
    ```excel
    =LOWER(TRIM(LEFT(RIGHT(G3, LEN(G3) - FIND("/", G3)), IFERROR(FIND(" por ", RIGHT(G3, LEN(G3) - FIND("/", G3))) - 1, LEN(RIGHT(G3, LEN(G3) - FIND("/", G3)))))))
    ```

### 3. Estandarización de Tipo de Trámite
Esto lo realicé utilizando la función de buscar y reemplazar tiempo antes de empezar este proyecto. De haberlo hecho ahora, hubiera utilizado un diccionario como hice con los delitos.

### 4. Estandarización de Delitos
- Eliminación de la tentativa (en el análisis no lo iba a utilizar) con la fórmula:
  ```excel
  =LOWER(TRIM(SUBSTITUTE(H3, "en tentativa", "")))
  ```
- Creación de una columna que indica si el delito es tentado o no:
  ```excel
  =IF(OR(ISNUMBER(SEARCH("tva", G3)), ISNUMBER(SEARCH("tentativa", G3))), "si", "no")
  ```
- Normalización de texto a minúsculas y eliminación de espacios adicionales utilizando funciones generales de Excel.
- Utilización de listado de delitos estandarizados: Se copió la columna de delitos y se eliminaron duplicados. Luego se creó otra columna con los valores que debían utilizarse y se aplicó:
  ```excel
  =IFERROR(VLOOKUP(A25, $B$2:$C$147, 2, FALSE), "NOT FOUND")
  ```

### 5. Exportación Final
Los datos procesados se guardaron en `Dataset juzgado.csv` para su uso posterior.

## Análisis y Visualizaciones en Power BI

El archivo `Estadistica ingresos 2020-2024.pbix` contiene un análisis de los datos procesados. A continuación, se describe el proceso realizado en Power BI:

### 1. Importación de Datos
- Se utilizó el archivo `Dataset juzgado.csv` como fuente principal de datos.

### 2. Limpieza y Transformación en Power Query
- Se ajustaron los tipos de datos (fechas, texto y números).
- Se filtraron registros irrelevantes para el análisis, como delitos con valores nulos o categorías no consideradas.
- se ilimaron columnas que no iban a usarse
- Se agregaron columnas calculadas:
  - **es delito X**: Para clasificar los delitos según su naturaleza

### 3. Medidas Calculadas
Se crearon las siguientes medidas en Power BI para facilitar el análisis:

- **Total de ingresos por año**:
  - con la fórmula  COUNTROWS('Dataset juzgado')
  
- **total de causas de tipo X**:
  
  - con la fórmula COUNTROWS(FILTER('Dataset juzgado', 'Dataset juzgado'[es delito sexual] = TRUE()))
  
- **total de tramites X**:
 
  - Con la fórmula: CALCULATE(COUNTROWS('Dataset juzgado'), NOT (CONTAINSSTRING('Dataset juzgado'[tipo trámite], "sjp") || CONTAINSSTRING('Dataset juzgado'[tipo trámite], "elevación") || CONTAINSSTRING('Dataset juzgado'[tipo trámite], "sobreseimiento") || CONTAINSSTRING('Dataset juzgado'[tipo trámite], "allanamiento") || CONTAINSSTRING('Dataset juzgado'[tipo trámite], "seguridad")|| CONTAINSSTRING('Dataset juzgado'[tipo trámite], "competencia")|| CONTAINSSTRING('Dataset juzgado'[tipo trámite], "amparo")))
Se muestra esa formula que es la mas larga, para cada categoría especifica se utilizó el CONTAINSTRING que corespondia

### 4. Visualizaciones
Las principales visualizaciones incluyen:
- **Gráficos de lineas**: Distribución de causas ingresadas por año
- **Tarjetas**: Para señalar el número de causas ingresdadas, delitos, etc.
- **Gráficos de columnas**: Distribución de causas por empleado
- **Gráficos circulares**: Distribución de trámites y distribución de delitos sexuales.

### 5. Exportación y Uso
- El reporte fue diseñado para responder preguntas específicas como:
  - ¿Cuáles son las tendencias de ingreso de causas por año?
  - ¿Cuál es la evolución de los delitos sexuales y las medidas cautelares solicitadas?
  - ¿Cuál es la tendencias en cuanto al por qué ingresan las causas al Juzgado?
  - ¿Cuál es la distribución interna de causas en el juzgado?
    
- El archivo `.pbix` está listo para ser reutilizado o ajustado según nuevas necesidades.

## Resultados Finales

- **Principales correcciones**:
  - Eliminación de cualquier vestigio de nombre propio.
  - Eliminación de columnas intrascendentes.
  - Estandarización de formatos en 2 columnas clave.
- **Visualización**
    [Dashboard interactivo](https://app.powerbi.com/view?r=eyJrIjoiZmNlNTgxOGQtMTcyNy00ZmNlLWJkM2ItNDYwNDg4OWYxMDE1IiwidCI6ImMyMjU5NDE5LWRiZGQtNDI5MC05ZWFmLWJhODNiZjQzNDkyNiIsImMiOjR9)


## Licencia
Este proyecto está licenciado bajo la licencia Creative Commons Zero. Consulta el archivo `LICENSE` para más información.

## Contacto
Para más información o dudas, puedes contactar al propietario del proyecto a través de emilioge@protonmail.com.


