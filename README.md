# EvalDoc-PDFGen
Script generador de libros PDF con datos de Evaluacion Docente

Name: EvalDoc-PDFGen

Version: 0.1

Dev: Ing. Hector Rafael Gonzalez Vega

Date: 21/11/2023

Update: ##/##/##

## Indice

* [Descripcion del proyecto](#descripcion-del-proyecto)
* [Estado del Proyecto](#estado-del-proyecto)
* [Tecnologias Utilizadas](#tecnologias-utilizadas)
* [Librerias Interna](#-librerias-interna)
* [Licecia](#licencia)
* [Versiones](#Versiones)
* [Codigo](#Codigo)

## Descripcion del proyecto
Script para automatizar la creacion de PDF para presentar los datos de la evaluacion docente.

El desarrollo de esta aplicacion se dio porque las fechas de evaluacion docente se habian acercado y el proyecto principal del mismo aun no estaba terminado, por lo que fue necesario generar un script para automatizar la generarcion de PDF's para dar formato y presentacion a la evaluacion docente.

Este script requiere de CSV obtenidos de google forms, mas concretamente, formularios generados con ***EvalDoc-FormGen*** y de los nombres a los que fueron ecaluados en el formulario descargado.

Dentro vienen ejemplos para hacer funcionar el script.

## Estado del Proyecto
Este Script se cuentra pausado.

## Tecnologias Utilizadas
+ cachetools        5.3.2
+ fpdf2             1.7.2

## Librerias Interna

### csv_tool
Se encarga de leer y contar las respuestas por profesor dentro del csv descargado desde un formulario de google.

### pdf_tool
Se encarga de generar el PDF con datos de las respues y graficos para una mejor presentacion de los mismos de la evaluacion docente.

## Licecia
Este Proyecto esta bajo Creative Commons CC-BY "Atribución 3.0 No portada" License:
https://creativecommons.org/licenses/by/3.0/deed.es

## Versiones

v0.1
--
Genera un PDF para dar formato a la entrega de datos de la evaluacion docente

## Codigo

### main.py

```python
#---------------------------------------------------------------------------------------
# Aplicacion generadora de libros excel con datos de Evaluacion Docente
#Name: RCD-Gen
#Version: 0.0.1
#Dev: Ing. Hector Rafael Gonzalez Vega
#Date: 21/11/2023
#update: ##/##/####
#note: Este script funciona con los csv descargados de las evaluaciones docentes generadas con EvalDoc-FormGen,
#      asi como otros script para leer y generar los pdf: csv_tool.py y pdf_tool.py
#---------------------------------------------------------------------------------------

from csv_tool import reader
from pdf_tool import generate

def main(group, teachers):
    '''
        Funcion principal

        Args:
            group (String): Nombre del grupo a evaluar. Este mismo debe ser el nombre del archivo csv sin espacios.

            teachers (List): Lista con los nombres de los profesores a evaluar
        
        Returns:
            None
    '''
    path = '/ruta/csv'
    fle = group.replace(' ','',)
    

    #--Las opciones de respuestas deben ser las mismas a las generadas en el formulario de google
    #answers = ['Desacuerdo','Mas o menos','De acuerdo']

    #--Las respuestas deben ser las mismas que las generadas 
    #questions = [
    #    'Pregunta 1',
    #    'Pregunta 2',
    #    'Pregunta 3',
    #    ]

    data = reader(f'csvFiles/{fle}.csv', teachers)
    generate(questions, data, teachers, answers, group, path)

#--Titulo del archivo csv asi como el que se usara para titulo de pdf
#title = 'Grupo'
#--Mismos nombres de profesores que fueron evaluados en el formulario de google y se encuentran en el csv.
#teachers = ['Alvarado','Flores','Guevara']

#main(title,teachers)
```

### csv_tool
```python
import csv
#import pprint

def reader(path,t, questions):
    '''
        Funcion que lee el archivo csv

        Args:
            path (String): Nombre de la ruta del archivo

            t (List): Lista con los nombres de los profesores a evaluar

            questions (List): Lista con las preguntas del formulario
        
        Returns:
            c (Dictionary): Diccionario obtenido de la funcion tool
    '''
    answers =[]
    with open(path, 'r', encoding='utf-8') as file:
        data = csv.DictReader(file)

        for dt in data:
            answers.append(dt)
            

    c = tool(answers,t, questions)

    return c

def tool(answers, teachers, questions):
    '''
        Funcion principal

        Args:
            answers (List): Lista de las respuestas

            teachers (List): Lista con los nombres de los profesores

            questions (List): Lista con las preguntas del formulario
        
        Returns:
            qstn (Dictionary): Diccionario con informacion desenglosada por profesor
    '''
    qstn ={}
    for teach in teachers:
        results = {}
        for quest in questions:
            T, R1, R2, R3 = 0, 0, 0, 0,
            c = []
            for dicc in answers:
                if quest == 'Comentarios para: ':
                    c.append(dicc[f"{quest}{teach}"])
                else:
                    if 'Desacuerdo' == dicc[f"{quest}[{teach}]"]:
                        R1 += 1
                        T +=1
                    elif 'Mas o menos' == dicc[f"{quest}[{teach}]"]:
                        R2 += 1
                        T +=1
                    elif 'De acuerdo' == dicc[f"{quest}[{teach}]"]:
                        R3 += 1
                        T +=1
                    
            if quest != 'Comentarios para: ':
                results[quest] = {'Desacuerdo':R1, 'Mas o menos':R2, 'De acuerdo':R3, 'Total':T}
            else:
                results['Comentarios'] = c

        qstn[teach] = results

    return qstn

```

### pdf_tool
```python
from fpdf import FPDF
from chart_tool import chart

def generate(questions, data, teachers, answers, group, path):
    '''
        Funcion encargada de generar el pdf

        Args:
            questions (List): Lista con las preguntas del formulario

            data (Dictionary): Diccionario con informacion desenglosada por profesor

            teachers (List): Lista con los nombres de los profesores

            answers (List): Lista de las respuestas

            group (String): Nombre titulo del pdf

            path (String): Ruta donde se guardara el pdf generado

        
        Returns:
            None
    '''
 
    
    class PDF(FPDF):
            '''
                Clase para generar el PDF, hijo de FPDF
            '''
        
        # Cabeza de pagina
        def header(self):
            # Logo
            self.image('logo.png', 10, 8, 40)
            # Arial bold 15
            self.set_font('Arial', 'B', 15)
            # Se mueve a la derecha
            self.cell(80)
            # titulo
            self.cell(0, 10, title, 0, 0, 'R')
            # Salto de linea
            self.ln(20)

        # Pie de pagina
        def footer(self):
            # Posicion a 1.5 cm desde abajo
            self.set_y(-15)
            # Arial italic 8
            self.set_font('Arial', 'I', 8)
            # Numeracion de pagina
            self.cell(0, 10, "Evaluacion Docente", 0, 0, 'C')

    for teacher in teachers:
        counter = 0
        title = f'Evaluacion Docente de {teacher}, grupo {group}'
        # Inicio del programa
        pdf = PDF(orientation = 'L',)
        pdf.alias_nb_pages()
        pdf.add_page()
        pdf.set_font('Times', 'B', 7.8)
        top = pdf.y #Se guarda la coordenada superior
        offset = pdf.x + 145 #Se calcula la posicion x del siguiente Cell
        pdf.multi_cell(145, 8, "Preguntas", 1,'C') #Se coloca el primer cell
        pdf.y = top #Se resetean las coordenadas
        pdf.x = offset 
        pdf.multi_cell(22, 4, "Desacuerdo", 1, 'C') #Se coloca el siguiente Cell
        pdf.y = top
        pdf.x = offset + 22
        pdf.multi_cell(22, 4, "Mas o menos", 1, 'C')
        pdf.y = top
        pdf.x = offset + 22 * 2
        pdf.multi_cell(22, 4, "De acuerdo", 1, 'C')
        pdf.y = top
        pdf.x = offset + 22 * 5
        pdf.multi_cell(22, 8, "Total", 1, 'C')

        percent = {}
        for r in answers:
            percent[r] = []

        for q in questions:
            counter +=1
            pdf.set_font('Times', '', 8)
            pdf.cell(145, 4, f"{counter}.-{q}", 1, 0)
            pdf.set_font('Times', 'B', 8)

            for r in answers:
                pdf.cell(11, 4, f'{data[teacher][q][r]}', 1, 0, 'C')
                pdf.cell(11, 4, f"{round((data[teacher][q][r]/data[teacher][q]['Total'])*100, 0)}%", 1, 0, 'C')
                percent[r].append(round((data[teacher][q][r]/data[teacher][q]['Total'])*100, 0))
            #Total
            pdf.cell(22, 4, f"{data[teacher][q]['Total']}", 1, 1, 'C')
        
        counter = 0
        pdf.ln(3)
        pdf.set_font('Times', 'B', 8, )
        pdf.cell(11, 4, f"Comentarios: {len(data[teacher]['Comentarios'])}", 0, 1, 'C')
        pdf.set_font('Times', '', )
        for c in data[teacher]['Comentarios']:
            counter +=1
            pdf.multi_cell(0, 4, f"{counter}.- {c.encode('latin-1', 'replace').decode('latin-1')}", 0, 1, 'C')
        
        pdf.add_page()
        pdf.set_font('Times', '', 8)
        
        # se genera los graficos
        chart(percent)
        pdf.image('test.jpg', 10, 50,270)

        flr = group.replace(' ','',)
        pdf.output(f"{path}/{flr}/Evaluacion_{teacher}.pdf", 'F')

        print(f"Se guardo PDF de: {teacher}")
```
