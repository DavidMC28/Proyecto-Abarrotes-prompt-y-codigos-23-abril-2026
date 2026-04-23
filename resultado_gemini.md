¡Hola! Como desarrollador senior, me encanta este desafío. Vamos a estructurar un plan de trabajo sólido para crear **AbarrotesCRUD**. 

Trabajaremos bajo una arquitectura limpia, integrando **Firebase** y utilizando **Antigravity** (que en el ecosistema de IA y desarrollo suele referirse a frameworks de orquestación de agentes o flujos de trabajo automatizados) para guiar a los estudiantes.

---

## 1. Estructura de Directorios y Preparación
Primero, organizamos el entorno local según tus requerimientos:

* **Ruta:** `xfluttermacias0633/crudabarrotes`
* **Comando de creación:**
    ```bash
    mkdir xfluttermacias0633
    cd xfluttermacias0633
    flutter create crudabarrotes
    ```

---

## 2. Configuración de Firebase (Consola y Librerías)
Para que el proyecto funcione, debemos vincularlo con la consola de Firebase.

### Paso A: Firestore
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea un proyecto llamado `AbarrotesCRUD`.
3. En el menú lateral, selecciona **Firestore Database** y crea una base de datos en "Modo de prueba".
4. Crea una colección llamada `productos`.

### Paso B: `pubspec.yaml`
Para integrar los servicios, añade estas dependencias. Presta atención a la indentación:

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0      # Librería base
  cloud_firestore: ^5.0.0    # Para la base de datos
  google_fonts: ^6.2.1       # Para estética visual
```

---

## 3. Metodología de Agentes y Flujo de Trabajo (Práctica Guiada)
Para esta práctica con **Antigravity**, definiremos roles que los estudiantes deben asumir o que la IA de apoyo ejecutará:

| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect** | Diseñador de Estructura | Define modelos de datos y carpetas. |
| **Firebase Handler** | Especialista en Datos | Gestiona la conexión y el flujo de Firestore. |
| **UI Designer** | Desarrollador Frontend | Crea widgets atractivos y manejo de estados. |

### Estructura de Carpetas Sugerida:
```text
lib/
├── agents/          # Lógica de flujo de trabajo
├── models/          # Modelo de Producto
├── services/        # Consultas a Firebase (CRUD)
├── views/           # Pantallas (Listado, Formulario)
└── main.dart        # Punto de entrada
```

---

## 4. Implementación del Código Funcional

### Modelo de Datos (`lib/models/producto_model.dart`)
```dart
class Producto {
  String id;
  String nombre;
  double precio;
  int stock;

  Producto({required this.id, required this.nombre, required this.precio, required this.stock});

  // Convertir de Firestore a Objeto
  factory Producto.fromMap(Map<String, dynamic> data, String id) {
    return Producto(
      id: id,
      nombre: data['nombre'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
      stock: data['stock'] ?? 0,
    );
  }

  // Convertir de Objeto a JSON para Firestore
  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'precio': precio,
      'stock': stock,
    };
  }
}
```

### Servicio CRUD (`lib/services/firebase_service.dart`)
Aquí implementamos los 4 pilares: **C**reate, **R**ead, **U**pdate, **D**elete.

```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/producto_model.dart';

class FirebaseService {
  final CollectionReference collection = FirebaseFirestore.instance.collection('productos');

  // LEER (Stream para actualización en tiempo real)
  Stream<List<Producto>> getProductos() {
    return collection.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Producto.fromMap(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // CREAR
  Future<void> addProducto(Producto p) => collection.add(p.toMap());

  // ACTUALIZAR
  Future<void> updateProducto(Producto p) => collection.doc(p.id).update(p.toMap());

  // BORRAR
  Future<void> deleteProducto(String id) => collection.doc(id).delete();
}
```

### Interfaz de Usuario (`lib/views/home_view.dart`)
Utilizaremos colores vibrantes como **Indigo** y **Amber** para un look profesional.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/producto_model.dart';

class HomeView extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Abarrotes CRUD', style: TextStyle(color: Colors.white)),
        backgroundColor: Colors.indigoAccent,
      ),
      body: StreamBuilder<List<Producto>>(
        stream: _service.getProductos(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          
          return ListView.builder(
            itemCount: snapshot.data!.length,
            itemBuilder: (context, index) {
              final prod = snapshot.data![index];
              return Card(
                elevation: 4,
                margin: EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                child: ListTile(
                  leading: CircleAvatar(backgroundColor: Colors.amber, child: Text('\$')),
                  title: Text(prod.nombre, style: TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text('Precio: ${prod.precio} | Stock: ${prod.stock}'),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => _mostrarForm(context, prod)),
                      IconButton(icon: Icon(Icons.delete, color: Colors.red), onPressed: () => _service.deleteProducto(prod.id)),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.indigoAccent,
        child: Icon(Icons.add, color: Colors.white),
        onPressed: () => _mostrarForm(context, null),
      ),
    );
  }

  void _mostrarForm(BuildContext context, Producto? p) {
    final nameCtrl = TextEditingController(text: p?.nombre ?? '');
    final priceCtrl = TextEditingController(text: p?.precio.toString() ?? '');
    final stockCtrl = TextEditingController(text: p?.stock.toString() ?? '');

    showModalBottomSheet(
      context: context,
      isScrollControlled: true,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 20, right: 20, top: 20),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nameCtrl, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: priceCtrl, decoration: InputDecoration(labelText: 'Precio'), keyboardType: TextInputType.number),
            TextField(controller: stockCtrl, decoration: InputDecoration(labelText: 'Stock'), keyboardType: TextInputType.number),
            SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.indigoAccent),
              child: Text(p == null ? 'Crear' : 'Actualizar', style: TextStyle(color: Colors.white)),
              onPressed: () {
                final nuevo = Producto(
                  id: p?.id ?? '',
                  nombre: nameCtrl.text,
                  precio: double.parse(priceCtrl.text),
                  stock: int.parse(stockCtrl.text),
                );
                p == null ? _service.addProducto(nuevo) : _service.updateProducto(nuevo);
                Navigator.pop(context);
              },
            ),
            SizedBox(height: 20),
          ],
        ),
      ),
    );
  }
}
```

### Inicialización Principal (`lib/main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'views/home_view.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización vital
  runApp(MaterialApp(
    debugShowCheckedModeBanner: false,
    theme: ThemeData(primarySwatch: Colors.indigo),
    home: HomeView(),
  ));
}
```

---

## 5. Guía para Estudiantes (Antigravity Flow)
1.  **Fase de Análisis (Agente Architect):** El alumno dibuja la estructura de la colección en papel antes de entrar a la consola.
2.  **Fase de Conexión (Agente Firebase):** El alumno debe verificar que el archivo `google-services.json` esté en la carpeta `android/app`.
3.  **Fase de Construcción (Agente UI):** Implementación de los widgets `StreamBuilder` para que vean la magia de la reactividad: si cambias un dato en la consola de Firebase, ¡se actualiza solo en el celular!

Este flujo asegura que no solo copien código, sino que entiendan la jerarquía de una aplicación profesional.
