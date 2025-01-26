import cv2
import face_recognition
import sqlite3
from datetime import datetime

# Conectar a la base de datos
conn = sqlite3.connect('asistencia.db')
cursor = conn.cursor()

# Crear tabla de asistencia si no existe
cursor.execute('''
CREATE TABLE IF NOT EXISTS asistencia (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    fecha TEXT NOT NULL,
    hora TEXT NOT NULL
)
''')
conn.commit()

# Cargar imágenes conocidas y sus nombres
known_face_encodings = []
known_face_names = []

# Agregar imágenes y nombres de estudiantes
# Ejemplo:
# imagen = face_recognition.load_image_file("ruta/a/la/imagen.jpg")
# encoding = face_recognition.face_encodings(imagen)[0]
# known_face_encodings.append(encoding)
# known_face_names.append("Nombre del Estudiante")

# Inicializar la cámara
video_capture = cv2.VideoCapture(0)

while True:
    # Capturar un solo frame de video
    ret, frame = video_capture.read()

    # Convertir la imagen de BGR (OpenCV) a RGB (face_recognition)
    rgb_frame = frame[:, :, ::-1]

    # Encontrar todas las caras y sus codificaciones en el frame actual
    face_locations = face_recognition.face_locations(rgb_frame)
    face_encodings = face_recognition.face_encodings(rgb_frame, face_locations)

    for face_encoding in face_encodings:
        # Verificar si la cara es de alguien conocido
        matches = face_recognition.compare_faces(known_face_encodings, face_encoding)
        name = "Desconocido"

        # Si se encuentra una coincidencia
        if True in matches:
            first_match_index = matches.index(True)
            name = known_face_names[first_match_index]

            # Registrar asistencia en la base de datos
            now = datetime.now()
            fecha = now.strftime("%Y-%m-%d")
            hora = now.strftime("%H:%M:%S")
            cursor.execute("INSERT INTO asistencia (nombre, fecha, hora) VALUES (?, ?, ?)", (name, fecha, hora))
            conn.commit()

        # Dibujar un cuadro alrededor de la cara
        for (top, right, bottom, left) in face_locations:
            cv2.rectangle(frame, (left, top), (right, bottom), (0, 0, 255), 2)
            cv2.putText(frame, name, (left + 6, bottom - 6), cv2.FONT_HERSHEY_DUPLEX, 0.5, (255, 255, 255), 1)

    # Mostrar el frame resultante
    cv2.imshow('Video', frame)

    # Salir con la tecla 'q'
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Liberar la cámara y cerrar ventanas
video_capture.release()
cv2.destroyAllWindows()
conn.close()
