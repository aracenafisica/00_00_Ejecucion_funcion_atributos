![](imagenes/UC_FMRI.jpg)

---

---

***Andres Eduardo Aracena Rangel***

*Estudiante del programa del Magister en Física Médica*

---

---

El siguiente Script de Python forma parte del trabajo especial de grado.

Profesora Guía:

*PhD María Daniela Cornejo*

---

---

Imagenes de fMRI extraidas de OpenNeuro:

- [ds001454](https://openneuro.org/datasets/ds001454/versions/1.3.1)
- [ds002422](https://openneuro.org/datasets/ds002422/versions/1.1.0)
- [ds004101](https://openneuro.org/datasets/ds004101/versions/1.0.1)

---

---

Con referencia:

- [Pagina oficial NIbabel](https://nipy.org/nibabel/index.html)

---

---

 # Aplicación de la función 'atributos_img'

Aplicamos la función *atributos_img* a tres imagenes distintas (imagenes de los estudios ds001454, ds002422 y ds4101 de OpenNeuro)

## Importamos librerias



```python
import time # medir el tiempo de ejecución de nuestros programas
start = time.process_time()
inicio = time.time()
```


```python
import os # El módulo os nos permite acceder a funcionalidades dependientes del Sistema Operativo
from os.path import join as opj # Este método concatena varios componentes de ruta con exactamente un separador de directorio(‘/’)

import nibabel as nib # Acceso de letcura/escritura y visualización de algunos formatos comunes de neuroimagen
```

## Definimos Parametros 


```python
'''
Ruta del directorio de la data
'''
path_data = '/home/aracena/data/'

'''
Ruta donde reposa las imágenes anatómicas
'''
# Estudio ds001454
path_ds001454 = opj(path_data,'ds001454','sub-01','ses-1')
path_ana_ds001454 = opj(path_ds001454, 'anat','sub-01_ses-1_T1w.nii.gz')
#sub-01_ses-1_T1w.json

# Estudio ds002422
path_ds002422 = opj(path_data,'ds002422','sub-01')
path_ana_ds002422 = opj(path_ds002422, 'anat', 'sub-01_T1w.nii')
#T1w.json

# Estudio ds004101
path_ds004101 = opj(path_data,'ds004101','sub-09114','ses-1pre')
path_ana_ds004101 = opj(path_ds004101,'anat', 'sub-09114_ses-1pre_T1w.nii.gz')
#T1w.json

'''
Ruta donde reposa las imágenes funcional
'''
# Estudio ds001454
path_fis_ds001454 = opj(path_ds001454, 'func','sub-01_ses-1_task-rest_run-01_bold.nii.gz')

# Estudio ds002422
path_fis_ds002422 = opj(path_ds002422, 'func', 'sub-01_task-rest_bold.nii.gz')

# Estudio ds004101
path_fis_ds004101 = opj(path_ds004101, 'func', 'sub-09114_ses-1pre_task-rest_bold.nii.gz')

'''
Ruta donde se guardaran los resultados
'''
path_expe = '/home/aracena/thesis_ds004101/00_fase0_tips_nibabel_funciones/'

path_output = opj(path_expe,'00_01_atributos_nibabel_and_func_atributoimg', 'output')

# Crear la(s) carpeta(s) de salida
os.system('mkdir -p %s'%path_output);
```

## Función 'atributos_img'


```python
'''
Función para extraer los atributos de la(s) imagen(es).

Inputs:

- img: Diccinario con las imagenes nifti

Output:

df_atri: Dataframe con los principales atributos de las imagenes contenidas en el diccionario 'img'
'''

def atributo_img(img):
    import pandas as pd
    lista_img = list(img.keys()) # Creamos una lista con las claves del diccionario

    df_atri = pd.DataFrame()
    df_atri.index = ['forma', 'dimension', 'orientacion', '', 'x_img', 'y_img' , 'z_img', 'volumenes_(N)', 'voxel_size_(mm)', 
                       'TR_(s)', 'tipo_dato', 'numero_voxels','min_img', 'max_img']
    
    for i, ima in enumerate(lista_img):
        lista_atri = []
        #cargamos el header de la imagen
        header_img = img[ima].header
        
        # Forma y dimensión de la imágen
        forma = header_img.get_data_shape() 
        lista_atri.append(str(forma))
        lista_atri.append(len(forma))
        
        # Orientacion de la imágen
        orientacion = nib.orientations.aff2axcodes(img[ima].affine)
        lista_atri.append(orientacion)

        # x, y, z y volumenes
        ejes=[]
        for i in range(len(orientacion)):
            tam = img[ima].shape[i]
            ori = str(orientacion[i])
            if  ori == 'L'or ori == 'R':
                x_img = tam
                a = 'x'

            elif ori == 'A' or ori == 'P':
                y_img = tam
                a = 'y'

            elif ori == 'I'or ori == 'S':
                z_img = tam
                a = 'z'
                
            ejes.append(a)
        
        # Agregamos a la lista de atributos forma, x, y, z
        lista_atri.append(ejes)
        lista_atri.append(x_img)
        lista_atri.append(y_img)
        lista_atri.append(z_img)
        
        # Agregamos volumes a la lista de atributos 
        if len(forma) == 4:
            lista_atri.append(forma[-1])
        else:
            lista_atri.append('1')

        # Tamaño del voxel
        tavo = header_img.get_zooms()[0:3]
        
        tamvox=[]
        for i in range(len(tavo)):
            tamvox.append(round(tavo[i],3))
            
        lista_atri.append(tamvox) 
        
        # Tiempo de exploración
        if len(header_img.get_zooms()) == 4:
            lista_atri.append(header_img.get_zooms()[-1])
        else:
            lista_atri.append('---')     
        
        
        #lista_atri.append(header_img.get_zooms()[-1])   # Tiempo de exploración
        lista_atri.append(header_img.get_data_dtype())   # Tipo de datos numérico
        lista_atri.append(img[ima].get_fdata().size) # Número de elementos de la matriz
        lista_atri.append(round(img[ima].get_fdata().min(),2)) # Valor minimo de la imágen
        lista_atri.append(round(img[ima].get_fdata().max(),2)) # Valor maximo de la imágen
        
        # Creamos DF de atributos de la imagen
        df_at = pd.DataFrame()
        df_at = pd.DataFrame(lista_atri)
        df_at.columns = [ima]
        df_at.index = df_atri.index
        #display(df_at)

        # Unimos las DF
        df_atri = pd.merge(df_atri, df_at,
                           right_index=True,left_index=True)
    return df_atri
```

&nbsp;
## Cargamos imagenes anatomica y fisiologica

### Creamos diccionario para cada estudio


```python
dic_ds001454 = {'anatomica_ds001454': nib.load(path_ana_ds001454), 
                'funcional_ds001454': nib.load(path_fis_ds001454)}
```


```python
dic_ds002422= {'anatomica_ds002422': nib.load(path_ana_ds002422), 
                'funcional_ds002422': nib.load(path_fis_ds002422)}
```


```python
dic_ds004101= {'anatomica_ds004101': nib.load(path_ana_ds004101), 
                'funcional_ds004101': nib.load(path_fis_ds004101)}
```

### Creamos lista con los diccionarios


```python
lista_dic = [dic_ds001454, dic_ds002422, dic_ds004101]
```

## Ejecutamos función


```python
for i,lidic in enumerate(lista_dic):
    atributo_imagenes = atributo_img(img= lista_dic[i])
    print('\n------------------------------------------------------------\n')
    display(atributo_imagenes)
    print('\n------------------------------------------------------------\n')
```

    
    ------------------------------------------------------------
    



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>anatomica_ds001454</th>
      <th>funcional_ds001454</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>forma</th>
      <td>(176, 256, 256)</td>
      <td>(64, 64, 36, 195)</td>
    </tr>
    <tr>
      <th>dimension</th>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>orientacion</th>
      <td>(R, A, S)</td>
      <td>(L, A, S)</td>
    </tr>
    <tr>
      <th></th>
      <td>[x, y, z]</td>
      <td>[x, y, z]</td>
    </tr>
    <tr>
      <th>x_img</th>
      <td>176</td>
      <td>64</td>
    </tr>
    <tr>
      <th>y_img</th>
      <td>256</td>
      <td>64</td>
    </tr>
    <tr>
      <th>z_img</th>
      <td>256</td>
      <td>36</td>
    </tr>
    <tr>
      <th>volumenes_(N)</th>
      <td>1</td>
      <td>195</td>
    </tr>
    <tr>
      <th>voxel_size_(mm)</th>
      <td>[1.0, 1.0, 1.0]</td>
      <td>[3.0, 3.0, 3.0]</td>
    </tr>
    <tr>
      <th>TR_(s)</th>
      <td>---</td>
      <td>2.0</td>
    </tr>
    <tr>
      <th>tipo_dato</th>
      <td>float32</td>
      <td>int16</td>
    </tr>
    <tr>
      <th>numero_voxels</th>
      <td>11534336</td>
      <td>28753920</td>
    </tr>
    <tr>
      <th>min_img</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>max_img</th>
      <td>1596.0</td>
      <td>2022.0</td>
    </tr>
  </tbody>
</table>
</div>


    
    ------------------------------------------------------------
    
    
    ------------------------------------------------------------
    



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>anatomica_ds002422</th>
      <th>funcional_ds002422</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>forma</th>
      <td>(256, 256, 176)</td>
      <td>(64, 64, 36, 200)</td>
    </tr>
    <tr>
      <th>dimension</th>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>orientacion</th>
      <td>(P, S, R)</td>
      <td>(L, A, S)</td>
    </tr>
    <tr>
      <th></th>
      <td>[y, z, x]</td>
      <td>[x, y, z]</td>
    </tr>
    <tr>
      <th>x_img</th>
      <td>176</td>
      <td>64</td>
    </tr>
    <tr>
      <th>y_img</th>
      <td>256</td>
      <td>64</td>
    </tr>
    <tr>
      <th>z_img</th>
      <td>256</td>
      <td>36</td>
    </tr>
    <tr>
      <th>volumenes_(N)</th>
      <td>1</td>
      <td>200</td>
    </tr>
    <tr>
      <th>voxel_size_(mm)</th>
      <td>[0.977, 0.977, 1.0]</td>
      <td>[3.594, 3.594, 3.78]</td>
    </tr>
    <tr>
      <th>TR_(s)</th>
      <td>---</td>
      <td>3.56</td>
    </tr>
    <tr>
      <th>tipo_dato</th>
      <td>float32</td>
      <td>int16</td>
    </tr>
    <tr>
      <th>numero_voxels</th>
      <td>11534336</td>
      <td>29491200</td>
    </tr>
    <tr>
      <th>min_img</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>max_img</th>
      <td>3080.0</td>
      <td>1725.0</td>
    </tr>
  </tbody>
</table>
</div>


    
    ------------------------------------------------------------
    
    
    ------------------------------------------------------------
    



<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>anatomica_ds004101</th>
      <th>funcional_ds004101</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>forma</th>
      <td>(256, 256, 176)</td>
      <td>(64, 64, 36, 189)</td>
    </tr>
    <tr>
      <th>dimension</th>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>orientacion</th>
      <td>(P, S, R)</td>
      <td>(L, A, S)</td>
    </tr>
    <tr>
      <th></th>
      <td>[y, z, x]</td>
      <td>[x, y, z]</td>
    </tr>
    <tr>
      <th>x_img</th>
      <td>176</td>
      <td>64</td>
    </tr>
    <tr>
      <th>y_img</th>
      <td>256</td>
      <td>64</td>
    </tr>
    <tr>
      <th>z_img</th>
      <td>256</td>
      <td>36</td>
    </tr>
    <tr>
      <th>volumenes_(N)</th>
      <td>1</td>
      <td>189</td>
    </tr>
    <tr>
      <th>voxel_size_(mm)</th>
      <td>[0.977, 0.977, 1.0]</td>
      <td>[3.0, 3.0, 3.75]</td>
    </tr>
    <tr>
      <th>TR_(s)</th>
      <td>---</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>tipo_dato</th>
      <td>float32</td>
      <td>int32</td>
    </tr>
    <tr>
      <th>numero_voxels</th>
      <td>11534336</td>
      <td>27869184</td>
    </tr>
    <tr>
      <th>min_img</th>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>max_img</th>
      <td>1547.0</td>
      <td>1957.0</td>
    </tr>
  </tbody>
</table>
</div>


    
    ------------------------------------------------------------
    


## Tiempo de ejecución


```python
fin = time.time()
end = time.process_time()
tiempo = fin - inicio
tiempo2 = end - start

print('--------------------------------------')
print('tiempo de ejecución\n\n', round(tiempo,3), 'seg\n', round(tiempo/60,3), 'min')     
print('--------------------------------------')
print('tiempo de ejecución del sistema y CPU\n\n', round(tiempo2,3), 'seg\n', round(tiempo2/60,3), 'min')
print('--------------------------------------')
```

    --------------------------------------
    tiempo de ejecución
    
     2.76 seg
     0.046 min
    --------------------------------------
    tiempo de ejecución del sistema y CPU
    
     2.667 seg
     0.044 min
    --------------------------------------


## Fin
