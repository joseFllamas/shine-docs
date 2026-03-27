# Descripción del proyecto

La intención de este proyecto es crear un aplicativo para personas con dislexia, que les ayude a mejorar su lectura y escritura.

El planteamiento inicial era hacerlo mediante un drupal10, pero he decidido cambiar a un enfoque más moderno y escalable. La idea es crear una aplicación web progresiva (PWA) que sea accesible desde cualquier dispositivo, ya sea móvil, tablet u ordenador.

Nos gustaría usar API REST de drupal para la comunicación entre el frontend y el backend, con un frontent desacoplado.

Los puntos fuertes de este proyecto son:

- Actividades para mejorar la lectura y escritura
- Monitorización gráfica de la evolución del usuario
- Actividades personalizadas según el nivel del usuario
- Comparativa de resultados entre sesiones
- Comparativa de resultados vs un resultado "estándar"
- Actividades con audio y texto
- Actividades con imágenes y texto
- Actividades con micrófono para medir el tiempo de lectura

Quiero que el proyecto sea escalable, por lo el proyecto debe permitir definir nuevos tipos de actividades, nuevos tipos de métricas, etc.  

Proponnos una arquitectura para este proyecto, qué tecnologías a usar y cómo lo implementarías, tanto para la parte de actividades como para los gráficos y estadísticas.

El diseño técnico de las actividades debe aprovechar todas aquellas partes que son comunes (introducción, mensaje de apoyo, etc.) y separar la lógica de cada tipo de actividad, para un máxima escalabilidad.

Valorar si drupal es la mejor opción para este proyecto o si hay alguna otra tecnología que se adapte mejor a las necesidades del proyecto.

Pasar a drupal11 antes de la implementación del frontend desacoplado.