<<<<<<< HEAD
from OpenGL.GL import *
from OpenGL.GLU import gluPerspective, gluLookAt
import glfw
import cv2
import numpy as np
from PIL import Image

# EL CUADRO VERDE CONTROLA LA ROTACION
# EL CUADRO AZUL CONTROLA LA TRASLACION 
# EL CUADRO ROJO CONTROLA EL ESCALAMIENTO,PONIENDO LA MANO EN LE LADO IZQUIERO SE 

rotation_angle = 0.0
translation_x = 0.0
translation_y = 0.0
scale_factor = 1.0
prev_gray = None
flow_threshold = 0.5
textures = {}

def load_texture(file_path):
    """Carga una textura desde un archivo de imagen."""
    try:
        img = Image.open(file_path)
        img = img.transpose(Image.FLIP_TOP_BOTTOM)  # Invertir verticalmente para OpenGL
        img_data = np.array(img, dtype=np.uint8)
    except FileNotFoundError:
        print(f"No se encontró el archivo de textura: {file_path}")
        sys.exit()

    texture_id = glGenTextures(1)
    glBindTexture(GL_TEXTURE_2D, texture_id)
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, img.width, img.height, 0, GL_RGB, GL_UNSIGNED_BYTE, img_data)

    # Configuración de la textura
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

    return texture_id

def init():
    glClearColor(0.5, 0.8, 1.0, 1.0)
    glEnable(GL_DEPTH_TEST)
    glEnable(GL_TEXTURE_2D)  # Activar texturas
    glMatrixMode(GL_PROJECTION)
    gluPerspective(60, 1.0, 0.1, 100.0)
    glMatrixMode(GL_MODELVIEW)

    # Cargar texturas
    global textures
    wall_texture_path = "Texturasdecasa.jpg"  # Especifica el título de la imagen aquí
    floor_texture_path = "Texturadesuelo.jpg"   # Especifica el título de la imagen del piso aquí
    roof_texture_path = "Texturadetecho.jpg"   # Especifica el título de la imagen del techo aquí

    textures['wall'] = load_texture(wall_texture_path)
    textures['floor'] = load_texture(floor_texture_path)
    textures['roof'] = load_texture(roof_texture_path)

def draw_cube():
    """Dibuja un cubo con texturas en las paredes."""
    glBindTexture(GL_TEXTURE_2D, textures['wall'])
    glBegin(GL_QUADS)

    # Frente
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, 1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, 1)

    # Atrás
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, -1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, -1)

    # Izquierda
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(-1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, -1)

    # Derecha
    glTexCoord2f(0.0, 0.0)
    glVertex3f(1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(1, 1, -1)

    glEnd()

def draw_roof():
    """Dibuja un techo con textura."""
    glBindTexture(GL_TEXTURE_2D, textures['roof'])
    glBegin(GL_TRIANGLES)

    # Frente
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Atrás
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Izquierda
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Derecha
    glTexCoord2f(0.0, 0.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    glEnd()

def draw_ground():
    """Dibuja un suelo con textura."""
    glBindTexture(GL_TEXTURE_2D, textures['floor'])
    glBegin(GL_QUADS)

    glTexCoord2f(0.0, 0.0)
    glVertex3f(-10, 0, 10)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(10, 0, 10)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(10, 0, -10)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-10, 0, -10)

    glEnd()

def draw_scene():
    global rotation_angle, translation_x, translation_y, scale_factor

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    glLoadIdentity()

    gluLookAt(4, 3, 8, 0, 1, 0, 0, 1, 0)

    glPushMatrix()
    glTranslatef(translation_x, translation_y, 0)
    glScalef(scale_factor, scale_factor, scale_factor)
    glRotatef(rotation_angle, 0, 1, 0)
    draw_ground()
    draw_cube()
    draw_roof()
    glPopMatrix()

    glfw.swap_buffers(window)

def process_camera():
    global rotation_angle, translation_x, translation_y, scale_factor, prev_gray

    cap = cv2.VideoCapture(0)
    ret, frame = cap.read()
    if not ret:
        cap.release()
        return

    height, width, _ = frame.shape
    roi_rot_top, roi_rot_bottom = 0, int(height * 0.3)
    roi_rot_left, roi_rot_right = 0, int(width * 0.3)
    roi_trans_top, roi_trans_bottom = 0, int(height * 0.3)
    roi_trans_left, roi_trans_right = int(width * 0.7), width
    roi_scale_top, roi_scale_bottom = int(height * 0.7), height
    roi_scale_left, roi_scale_right = 0, int(width * 0.3)

    prev_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    while not glfw.window_should_close(window):
        ret, frame = cap.read()
        if not ret:
            break

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

        roi_rot = gray[roi_rot_top:roi_rot_bottom, roi_rot_left:roi_rot_right]
        flow_rot = cv2.calcOpticalFlowFarneback(prev_gray[roi_rot_top:roi_rot_bottom, roi_rot_left:roi_rot_right],
                                                roi_rot, None, 0.5, 3, 15, 3, 5, 1.2, 0)
        flow_x_rot = np.mean(flow_rot[..., 0])

        roi_trans = gray[roi_trans_top:roi_trans_bottom, roi_trans_left:roi_trans_right]
        flow_trans = cv2.calcOpticalFlowFarneback(prev_gray[roi_trans_top:roi_trans_bottom, roi_trans_left:roi_trans_right],
                                                  roi_trans, None, 0.5, 3, 15, 3, 5, 1.2, 0)
        flow_x_trans = np.mean(flow_trans[..., 0])
        flow_y_trans = np.mean(flow_trans[..., 1])

        roi_scale = frame[roi_scale_top:roi_scale_bottom, roi_scale_left:roi_scale_right]
        gray_scale = cv2.cvtColor(roi_scale, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(gray_scale, (5, 5), 0)
        _, thresh = cv2.threshold(blurred, 60, 255, cv2.THRESH_BINARY_INV)
        contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        if contours:
            largest_contour = max(contours, key=cv2.contourArea)
            x, _, w, _ = cv2.boundingRect(largest_contour)
            hand_position = x + w / 2
            rec_width = roi_scale_right - roi_scale_left
            normalized_position = hand_position / rec_width

            if normalized_position > 0.6:
                scale_factor += 0.01
            elif normalized_position < 0.4:
                scale_factor -= 0.01

            scale_factor = max(0.5, min(2.0, scale_factor))

        if abs(flow_x_rot) > flow_threshold:
            rotation_angle += flow_x_rot * 5

        if abs(flow_x_trans) > flow_threshold:
            translation_x += flow_x_trans * 0.1
        if abs(flow_y_trans) > flow_threshold:
            translation_y -= flow_y_trans * 0.1

        prev_gray = gray

        cv2.rectangle(frame, (roi_rot_left, roi_rot_top), (roi_rot_right, roi_rot_bottom), (0, 255, 0), 2)
        cv2.rectangle(frame, (roi_trans_left, roi_trans_top), (roi_trans_right, roi_trans_bottom), (255, 0, 0), 2)
        cv2.rectangle(frame, (roi_scale_left, roi_scale_top), (roi_scale_right, roi_scale_bottom), (0, 0, 255), 2)

        cv2.imshow("Cámara", frame)

        draw_scene()
        glfw.poll_events()

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()

def main():
    global window

    if not glfw.init():
        sys.exit()

    width, height = 800, 600
    window = glfw.create_window(width, height, "Casa 3D con Rotación, Traslación y Escalamiento", None, None)
    if not window:
        glfw.terminate()
        sys.exit()

    glfw.make_context_current(window)
    glViewport(0, 0, width, height)
    init()
    process_camera()
    glfw.terminate()

if __name__ == "__main__":
    main()
=======
from OpenGL.GL import *
from OpenGL.GLU import gluPerspective, gluLookAt
import glfw
import cv2
import numpy as np
from PIL import Image

# EL CUADRO VERDE CONTROLA LA ROTACION
# EL CUADRO AZUL CONTROLA LA TRASLACION 
# EL CUADRO ROJO CONTROLA EL ESCALAMIENTO,PONIENDO LA MANO EN LE LADO IZQUIERO SE 

rotation_angle = 0.0
translation_x = 0.0
translation_y = 0.0
scale_factor = 1.0
prev_gray = None
flow_threshold = 0.5
textures = {}

def load_texture(file_path):
    """Carga una textura desde un archivo de imagen."""
    try:
        img = Image.open(file_path)
        img = img.transpose(Image.FLIP_TOP_BOTTOM)  # Invertir verticalmente para OpenGL
        img_data = np.array(img, dtype=np.uint8)
    except FileNotFoundError:
        print(f"No se encontró el archivo de textura: {file_path}")
        sys.exit()

    texture_id = glGenTextures(1)
    glBindTexture(GL_TEXTURE_2D, texture_id)
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, img.width, img.height, 0, GL_RGB, GL_UNSIGNED_BYTE, img_data)

    # Configuración de la textura
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR)
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR)

    return texture_id

def init():
    glClearColor(0.5, 0.8, 1.0, 1.0)
    glEnable(GL_DEPTH_TEST)
    glEnable(GL_TEXTURE_2D)  # Activar texturas
    glMatrixMode(GL_PROJECTION)
    gluPerspective(60, 1.0, 0.1, 100.0)
    glMatrixMode(GL_MODELVIEW)

    # Cargar texturas
    global textures
    wall_texture_path = "Texturasdecasa.jpg"  # Especifica el título de la imagen aquí
    floor_texture_path = "Texturadesuelo.jpg"   # Especifica el título de la imagen del piso aquí
    roof_texture_path = "Texturadetecho.jpg"   # Especifica el título de la imagen del techo aquí

    textures['wall'] = load_texture(wall_texture_path)
    textures['floor'] = load_texture(floor_texture_path)
    textures['roof'] = load_texture(roof_texture_path)

def draw_cube():
    """Dibuja un cubo con texturas en las paredes."""
    glBindTexture(GL_TEXTURE_2D, textures['wall'])
    glBegin(GL_QUADS)

    # Frente
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, 1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, 1)

    # Atrás
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, -1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, -1)

    # Izquierda
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(-1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-1, 1, -1)

    # Derecha
    glTexCoord2f(0.0, 0.0)
    glVertex3f(1, 0, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 0, 1)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(1, 1, -1)

    glEnd()

def draw_roof():
    """Dibuja un techo con textura."""
    glBindTexture(GL_TEXTURE_2D, textures['roof'])
    glBegin(GL_TRIANGLES)

    # Frente
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Atrás
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Izquierda
    glTexCoord2f(0.0, 0.0)
    glVertex3f(-1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(-1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    # Derecha
    glTexCoord2f(0.0, 0.0)
    glVertex3f(1, 1, -1)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(1, 1, 1)
    glTexCoord2f(0.5, 1.0)
    glVertex3f(0, 2, 0)

    glEnd()

def draw_ground():
    """Dibuja un suelo con textura."""
    glBindTexture(GL_TEXTURE_2D, textures['floor'])
    glBegin(GL_QUADS)

    glTexCoord2f(0.0, 0.0)
    glVertex3f(-10, 0, 10)
    glTexCoord2f(1.0, 0.0)
    glVertex3f(10, 0, 10)
    glTexCoord2f(1.0, 1.0)
    glVertex3f(10, 0, -10)
    glTexCoord2f(0.0, 1.0)
    glVertex3f(-10, 0, -10)

    glEnd()

def draw_scene():
    global rotation_angle, translation_x, translation_y, scale_factor

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    glLoadIdentity()

    gluLookAt(4, 3, 8, 0, 1, 0, 0, 1, 0)

    glPushMatrix()
    glTranslatef(translation_x, translation_y, 0)
    glScalef(scale_factor, scale_factor, scale_factor)
    glRotatef(rotation_angle, 0, 1, 0)
    draw_ground()
    draw_cube()
    draw_roof()
    glPopMatrix()

    glfw.swap_buffers(window)

def process_camera():
    global rotation_angle, translation_x, translation_y, scale_factor, prev_gray

    cap = cv2.VideoCapture(0)
    ret, frame = cap.read()
    if not ret:
        cap.release()
        return

    height, width, _ = frame.shape
    roi_rot_top, roi_rot_bottom = 0, int(height * 0.3)
    roi_rot_left, roi_rot_right = 0, int(width * 0.3)
    roi_trans_top, roi_trans_bottom = 0, int(height * 0.3)
    roi_trans_left, roi_trans_right = int(width * 0.7), width
    roi_scale_top, roi_scale_bottom = int(height * 0.7), height
    roi_scale_left, roi_scale_right = 0, int(width * 0.3)

    prev_gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

    while not glfw.window_should_close(window):
        ret, frame = cap.read()
        if not ret:
            break

        gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)

        roi_rot = gray[roi_rot_top:roi_rot_bottom, roi_rot_left:roi_rot_right]
        flow_rot = cv2.calcOpticalFlowFarneback(prev_gray[roi_rot_top:roi_rot_bottom, roi_rot_left:roi_rot_right],
                                                roi_rot, None, 0.5, 3, 15, 3, 5, 1.2, 0)
        flow_x_rot = np.mean(flow_rot[..., 0])

        roi_trans = gray[roi_trans_top:roi_trans_bottom, roi_trans_left:roi_trans_right]
        flow_trans = cv2.calcOpticalFlowFarneback(prev_gray[roi_trans_top:roi_trans_bottom, roi_trans_left:roi_trans_right],
                                                  roi_trans, None, 0.5, 3, 15, 3, 5, 1.2, 0)
        flow_x_trans = np.mean(flow_trans[..., 0])
        flow_y_trans = np.mean(flow_trans[..., 1])

        roi_scale = frame[roi_scale_top:roi_scale_bottom, roi_scale_left:roi_scale_right]
        gray_scale = cv2.cvtColor(roi_scale, cv2.COLOR_BGR2GRAY)
        blurred = cv2.GaussianBlur(gray_scale, (5, 5), 0)
        _, thresh = cv2.threshold(blurred, 60, 255, cv2.THRESH_BINARY_INV)
        contours, _ = cv2.findContours(thresh, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

        if contours:
            largest_contour = max(contours, key=cv2.contourArea)
            x, _, w, _ = cv2.boundingRect(largest_contour)
            hand_position = x + w / 2
            rec_width = roi_scale_right - roi_scale_left
            normalized_position = hand_position / rec_width

            if normalized_position > 0.6:
                scale_factor += 0.01
            elif normalized_position < 0.4:
                scale_factor -= 0.01

            scale_factor = max(0.5, min(2.0, scale_factor))

        if abs(flow_x_rot) > flow_threshold:
            rotation_angle += flow_x_rot * 5

        if abs(flow_x_trans) > flow_threshold:
            translation_x += flow_x_trans * 0.1
        if abs(flow_y_trans) > flow_threshold:
            translation_y -= flow_y_trans * 0.1

        prev_gray = gray

        cv2.rectangle(frame, (roi_rot_left, roi_rot_top), (roi_rot_right, roi_rot_bottom), (0, 255, 0), 2)
        cv2.rectangle(frame, (roi_trans_left, roi_trans_top), (roi_trans_right, roi_trans_bottom), (255, 0, 0), 2)
        cv2.rectangle(frame, (roi_scale_left, roi_scale_top), (roi_scale_right, roi_scale_bottom), (0, 0, 255), 2)

        cv2.imshow("Cámara", frame)

        draw_scene()
        glfw.poll_events()

        if cv2.waitKey(1) & 0xFF == ord('q'):
            break

    cap.release()
    cv2.destroyAllWindows()

def main():
    global window

    if not glfw.init():
        sys.exit()

    width, height = 800, 600
    window = glfw.create_window(width, height, "Casa 3D con Rotación, Traslación y Escalamiento", None, None)
    if not window:
        glfw.terminate()
        sys.exit()

    glfw.make_context_current(window)
    glViewport(0, 0, width, height)
    init()
    process_camera()
    glfw.terminate()

if __name__ == "__main__":
    main()
>>>>>>> 3bba82c553861060ba066a0c655667696621dfdc
