# Detección de nubes con imagenes Landsat

Primero es necesario crear una imagen docker con el archivo docker:

```
docker build -t python-fmask .
```

Una vez que la imagen es creada podemos usarla y descargarla para procesarla. Comenzaremos con un ejemplo simple. Descarga la imagen landsat, cambiamos nuestra carpeta de trabajo. Una vez que estemos en nuestro directorio deseado:
```
docker run -v $(pwd):/data python-fmask gsutil cp gs://earthengine-public/landsat/L7/026/046/LE70260462013004ASN00.tar.bz /data
```
Despues de extraemos el contenido del archivo zip, nos colocamos en el directorio creado y ejecutamos los siguientes comandos para crear una imagen con las nubes y sombras detectadas:

```
docker run -v $(pwd):/data python-fmask gdal_merge.py -separate -of HFA -co COMPRESSED=YES -o ref.img L*_B[1,2,3,4,5,7].TIF
docker run -v $(pwd):/data python-fmask gdal_merge.py -separate -of HFA -co COMPRESSED=YES -o thermal.img L*_B6_VCID_?.TIF
docker run -v $(pwd):/data python-fmask fmask_usgsLandsatSaturationMask.py -i ref.img -m *_MTL.txt -o saturationmask.img
docker run -v $(pwd):/data python-fmask fmask_usgsLandsatTOA.py -i ref.img -m *_MTL.txt -o toa.img
docker run -v $(pwd):/data python-fmask fmask_usgsLandsatStacked.py -t thermal.img -a toa.img -m *_MTL.txt -s saturationmask.img -o cloud.img
```
La mascara de la nube y sombra los encontrarás en tu directorio de trabajo con el nombre de cloud.img

## Windows

En Windows, Docker solo permite montar archivos en ciertos directorios uno de ellos es el que se señala aquí:

```

C:\\Users

```

Nuestra carpeta de trabajo deberá estar en el directorio descrito previamente. Por ejemplo, en nuestro caso usaremos una carpeta llamada "example":

```

C:\\Users\example

```
Una vez que estemos en la ruta, el comando cambia a:

```
docker run -v /$(pwd):/data python-fmask gsutil -cp gs://earthengine-public/landsat/L7/026/046/LE70260462013004ASN00.tar.bz /data
```
Cuando termines de descargar extrae el archivo con:

```
tar xvfj LE70260462013004ASN00.tar.bz
```
Ahora los archivos estan en el directorio:

```
docker run -v /$(pwd):/data python-fmask gdal_merge.py -separate -of HFA -co COMPRESSED=YES -o ref.img L*_B[1,2,3,4,5,7].TIF
docker run -v /$(pwd):/data python-fmask gdal_merge.py -separate -of HFA -co COMPRESSED=YES -o thermal.img L*_B6_VCID_?.TIF
docker run -v /$(pwd):/data python-fmask fmask_usgsLandsatSaturationMask.py -i ref.img -m *_MTL.txt -o saturationmask.img
docker run -v /$(pwd):/data python-fmask fmask_usgsLandsatTOA.py -i ref.img -m *_MTL.txt -o toa.img
docker run -v /$(pwd):/data python-fmask fmask_usgsLandsatStacked.py -t thermal.img -a toa.img -m *_MTL.txt -s saturationmask.img -o cloud.img
```

## NOTA

Puede ser el caso que la RAM no sea suficiente para este proceso, si es así, es necesario aumentar el tamaño de la memoria en la configuración de VirtualBox  se sugiere 4096Mb.
