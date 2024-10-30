import cv2 as cv
import numpy as np

# Cargar la imagen en escala de grises
img = cv.imread('descargas.png', 0)

# Tamaño de la imagen original
x, y = img.shape

# Factor de escala
scale = 2

# Crear una imagen escalada en blanco
scaled_img = np.zeros((int(x / scale), int(y / scale)), dtype=np.uint8)

# Escalar la imagen manualmente y oscurecerla
for i in range(0, x, scale):
    for j in range(0, y, scale):
        # Verificar que los índices no excedan los límites
        if i < x and j < y:
            # Asignar el valor del píxel a la nueva imagen escalada y oscurecerlo
            scaled_img[i // scale, j // scale] = img[i, j] // 2

# Aplicar convolución manual con un kernel 3x3 de valores 1/9
convoluted_img = np.zeros_like(scaled_img, dtype=np.uint8)
kernel = np.ones((3, 3), dtype=np.float32) / 9  # Matriz 3x3 con valores 1/9
kernel_size = kernel.shape[0]
offset = kernel_size // 2

# Convolución
for i in range(offset, scaled_img.shape[0] - offset):
    for j in range(offset, scaled_img.shape[1] - offset):
        sum_val = 0
        for ki in range(kernel_size):
            for kj in range(kernel_size):
                pixel_val = scaled_img[i + ki - offset, j + kj - offset]
                sum_val += pixel_val * kernel[ki, kj]
        convoluted_img[i, j] = np.clip(sum_val, 0, 255)

# Mostrar la imagen original, la imagen escalada y la imagen convolucionada
cv.imshow('Imagen Original', img)
cv.imshow('Imagen Escalada y Oscurecida', scaled_img)
cv.imshow('Imagen Convolucionada', convoluted_img)
cv.waitKey(0)
cv.destroyAllWindows()
