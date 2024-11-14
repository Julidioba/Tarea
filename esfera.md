import glfw
from OpenGL.GL import *
from OpenGL.GLU import gluNewQuadric, gluSphere, gluPerspective
import sys
import random
import math

# Variables globales para el ángulo de rotación y posición de la esfera
window = None
rotation_angle = 0.0  # Ángulo de rotación de la esfera
movement_offset_x = 0.0  # Offset para el movimiento en el eje X
movement_offset_y = 0.0  # Offset para el movimiento en el eje Y
movement_speed = 0.009  # Velocidad de movimiento
direction_angle = random.uniform(0, 2 * math.pi)  # Ángulo inicial aleatorio de movimiento

# Límite de los bordes
border_limit = 2.0

def init():
    glClearColor(0.0, 0.0, 0.0, 1.0)  # Fondo negro
    glEnable(GL_DEPTH_TEST)            # Activar prueba de profundidad
    glEnable(GL_LIGHTING)              # Activar iluminación
    glEnable(GL_LIGHT0)                # Activar la luz 0

    # Configuración de la perspectiva
    glMatrixMode(GL_PROJECTION)
    gluPerspective(45, 1.0, 0.1, 50.0)
    glMatrixMode(GL_MODELVIEW)

    # Configuración de la luz
    light_pos = [1.0, 1.0, 1.0, 0.0]  # Posición de la luz
    light_color = [1.0, 1.0, 1.0, 1.0]  # Color de la luz blanca
    ambient_light = [0.2, 0.2, 0.2, 1.0]  # Luz ambiental

    glLightfv(GL_LIGHT0, GL_POSITION, light_pos)
    glLightfv(GL_LIGHT0, GL_DIFFUSE, light_color)
    glLightfv(GL_LIGHT0, GL_AMBIENT, ambient_light)

    # Configuración de las propiedades de material
    material_diffuse = [1, 0.2, 1.0, 0.0]  # Color difuso (azul claro)
    glMaterialfv(GL_FRONT, GL_DIFFUSE, material_diffuse)

def draw_sphere(radius=1, slices=32, stacks=32):
    global rotation_angle, movement_offset_x, movement_offset_y

    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    glLoadIdentity()

    # Mover la esfera en la dirección calculada
    glTranslatef(movement_offset_x, movement_offset_y, -5)
    glRotatef(rotation_angle, 0, 1, 0)  # Rotar la esfera sobre su eje Y

    # Dibujar la esfera
    quadric = gluNewQuadric()
    gluSphere(quadric, radius, slices, stacks)

    glfw.swap_buffers(window)

def update_motion():
    global rotation_angle, movement_offset_x, movement_offset_y, direction_angle

    # Actualizar la posición de la esfera según el ángulo de dirección
    movement_offset_x += movement_speed * math.cos(direction_angle)
    movement_offset_y += movement_speed * math.sin(direction_angle)

    # Rebote en los bordes (izquierda, derecha, arriba, abajo)
    if movement_offset_x > border_limit:       # Limite derecho
        movement_offset_x = border_limit  # Ajustar la posición a los límites
        direction_angle = math.pi - direction_angle  # Invertir la dirección horizontal
    elif movement_offset_x < -border_limit:    # Limite izquierdo
        movement_offset_x = -border_limit  # Ajustar la posición a los límites
        direction_angle = math.pi - direction_angle  # Invertir la dirección horizontal

    if movement_offset_y > border_limit:       # Limite superior
        movement_offset_y = border_limit  # Ajustar la posición a los límites
        direction_angle = -direction_angle  # Invertir la dirección vertical
    elif movement_offset_y < -border_limit:    # Limite inferior
        movement_offset_y = -border_limit  # Ajustar la posición a los límites
        direction_angle = -direction_angle  # Invertir la dirección vertical

def main():
    global window

    # Inicializar GLFW
    if not glfw.init():
        sys.exit()
    
    # Crear ventana de GLFW
    width, height = 500, 500
    window = glfw.create_window(width, height, "Esfera en Movimiento y Rebote", None, None)
    if not window:
        glfw.terminate()
        sys.exit()

    glfw.make_context_current(window)
    glViewport(0, 0, width, height)
    init()

    # Bucle principal
    while not glfw.window_should_close(window):
        draw_sphere()
        update_motion()  # Actualizar el movimiento
        glfw.poll_events()

    glfw.terminate()

if __name__ == "__main__":
    main()
