# PhotoGallery

Hacer un aplicacion básica de cámara que almacene las fotos utilizando Ionic, Visual Studio Code y Android Studio

## Clonar
```bash
Git clone https://github.com/4lanPz/AM-PhotoGallery-2024A
```

## Pasos

- 1 Pre requisitos

Tener instalado Node.JS y Npm
Tener un IDE para customizar nuestro proyecto en este caso Visual Studio Code
Tener Android Studio configurado
Estos se pueden descargar desde la pagina web oficial de dependiendo de el OS que estes utilizando.

- 2 Empezar el proyecto

En este nombreproyecto es el nombre que le vamos a poner a nuestro proyecto, por lo que se puede poner el
que tu quieras para tu proyecto.

En este tambien debemos elegir que módulos vamos a ocupar, en este caso vamos a ocupar "NGModules"

Al finalizar no es necesario tener una cuenta de Ionic, asi que eso podemos indicar que no y con eso nuestro
proyecto se ha creado

- 3 Navegar al directorio e intalación dependencias

Utilizando la consola CMD podemos ir a el directorio de nuestro proyecto con 
```bash
cd nombreproyecto
```

Dentro de esta carpeta tendremos que instalar los módulos necesarios para que se ejecute nuestro proyecto:

```bash
npm install
```

- 4 Funcionalidades
Para que funcione correctamente las funciones de tomar y guardar las fotos en la galería, es necesario el servicio de camera y save picture

Para ello necesitaremos utilizar la API de Filesystem, en este se debe configurar la funcionalidades despues de instalar los capacitors con:
```bash
npm install @capacitor/camera
npm install @capacitor/filesystem
npm install @capacitor/splash-screen
```
Ahora ya con todo instalado correctamente podemos pasar a iniciar el servicio de la camara y el almacenamiento de las imagenes creando un directorio llamado services
```bash
npx ionic g service services/photo

```
Dentro de esta carpeta necesitaremos editar el archivo "Photo.service.ts" 
En este archivo vamos a codificar todos los datos necesarios para la funcionalidad de poder guardar datos y cargar datos.
Para ello necesitaremos crear clases async para poder guardar las fotos como addNewToGallery, que cada vez que tome una foto esta se podra guardar con un save Picture

```bash
// toma la foto
const capturedPhoto = await Camera.getPhoto({
      resultType: CameraResultType.Uri,
      source: CameraSource.Camera,
      quality: 100,
    });
// guarda la foto
    const savedImagedFile = await this.savePicture(capturedPhoto);

 // Actualiza las fotos por si hay nueva o elimina
    await Preferences.set({
      key: this.PHOTO_STORAGE,
      value: JSON.stringify(this.photos),
    });
```

```bash
// Guardar foto
private async savePicture(photo: Photo){
    // Convert photo to base64 format, required by Filesystem API to save
    const base64Data = await this.readAsBase64(photo);

    // Write the file to the data directory
    const fileName = Date.now() + '.jpeg';
    const savedFile = await Filesystem.writeFile({
      path: fileName,
      data: base64Data,
      directory: Directory.Data
    }) 

    // Use webPath to display the new image instead of base64 since it's already loaded into memory
    return{
      filepath: fileName,
      webviewPath: photo.webPath
    }
  }
```

```bash
// Carga la foto
public async loadSaved(){
    // Retrive cached photo array data
    const { value } = await Preferences.get({key: this.PHOTO_STORAGE});
    this.photos = (value ? JSON.parse(value) : []) as UserPhoto[];

    // Display the photo by reading into base64 format
    for (let photo of this.photos){
      // Read each saved photo's data from the Filesystem
      const readFile = await Filesystem.readFile({
        path: photo.filepath,
        directory: Directory.Data,
      })

      // Web platform only: Load the photo as base64 data
      photo.webviewPath = `data:image/jpeg;base64,$(readFile.data)`;
    }
  }
```
Estos códigos son los que nos permiten que en la clase addNewToGallery tenga las funcionalidades de tomar la foto y almacenarla, pero tambien vamos a codificar una funcion para poder eliminar las fotos 
```bash
// Borra la foto
public async deletePhoto(index: number) {
    const photoToRemove = this.photos[index];
  
    // Remove photo from array
    this.photos.splice(index, 1);
  
    // Remove photo file from filesystem
    await Filesystem.deleteFile({
      path: photoToRemove.filepath,
      directory: Directory.Data,
    });
  
    // Update storage
    await Preferences.set({
      key: this.PHOTO_STORAGE,
      value: JSON.stringify(this.photos),
    });
  }  
```

Ahora ya con todas las funcionalidades codificadas podemos editar el HTML para que esto funcione con un botón y que las imagenes se muestren.
En este caso vamos a utilizar la tab 2 para mostrar las imágenes, por lo que la vamos a editar para que muestre las imágenes.
```bash
<ion-header [translucent]="true">
  <ion-toolbar>
    <ion-title> Photo Gallery </ion-title>
  </ion-toolbar>
</ion-header>

<ion-content>
  <div>
    <ion-row>
      <ion-grid
        size="4"
        *ngFor="let Photo of photoService.photos; let i = index"
      >
        <ion-img [src]="Photo.webviewPath"></ion-img>
      </ion-grid>
    </ion-row>
    <ion-row>
      <ion-grid
        size="4"
        *ngFor="let Photo of photoService.photos; let i = index"
      >
        <ion-button (click)="deletePhoto(i)" color="danger">Delete</ion-button>
      </ion-grid>
    </ion-row>
  </div>
  <div>
    <ion-fab vertical="bottom" horizontal="center" slot="fixed">
      <ion-fab-button (click)="addPhotoToGallery()">
        <ion-icon name="camera"></ion-icon>
      </ion-fab-button>
    </ion-fab>
  </div>
</ion-content>

```
Con esto lo que hacemos es que las imagenes se vayan almacenando en un grid para que cada una vaya alojandose en un nuevo espacio y que no importe cuantas fotos haya, todas se muestren y asi mismo el boton de eliminar se muestre debajo de cada una de ellas
- 5 SplashScreen
Para hacer que nuestra aplicación tenga una imagen personalizada al momento de ejecutar la imagen en android necesitaremos hacer primero el build de la aplicacion con:
```bash
npx ionic build android
npx ionic capacitor open android
```
Ahora en nuestro entorno debe haberse creado una carpeta Android, en la cual deberemos ir a:
```bash
nombreproyecto/
├── Android/
│   ├── app/
│   │   ├── src/
|   |   |   ├── main/
|   |   |   |   ├── res/

```
Dentro de esta carpeta encontraremos varias carpetas que dentro de esta tienen un archivo llamado splash.png, para cambiarlo necesitaremos una imagen y reemplazar las que ya se encontraban en el los directorios
- 6 Ejecución
Para poder ejecutar nuestro proyecto simplemente necesitamos ejecutar el comando
```bash
Npx ionic s
```
Este comando ejecutará nuestra aplicación de forma web, entonces podremos comprobar que la aplicación es totalmente funcional.
si se quiere ejecutar en android se necesitan 2 comandos
```bash
npx ionic build android
npx ionic capacitor open android
```
El primero comando empezará a generar los archivos necesarios para que nuestra aplicación tambien funcione en android, loque genera una carpeta android.
El segundo comando lo que hace es abrir este proyecto de Android en Android Studio, esto para poder hacer la emulación de la aplicación en android o emulación directamente en nuestro dispositivo Android a través de Wifi

# Capturas
### Web
![image](https://github.com/4lanPz/AM-PhotoGallery-2024A/assets/117743495/8efa488c-bf9d-4057-870b-cd3b476359eb)

![image](https://github.com/4lanPz/AM-PhotoGallery-2024A/assets/117743495/19170dd5-0cdd-42e1-b18f-c358f5305dc6)

### Android
![image](https://github.com/4lanPz/AM-PhotoGallery-2024A/assets/117743495/28d0a761-4f09-4ded-9ae8-7ae7e356e6c3)

![image](https://github.com/4lanPz/AM-PhotoGallery-2024A/assets/117743495/6ad75273-b90a-4dd5-9bb3-d600f0d457b0)
