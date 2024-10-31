import cv2 as cv
import numpy as np

# Cargar la imagen en escala de grises
img = cv.imread('descargas.png', 0)

# Tamaño de la imagen original
x, y = img.shape

# Escala
scale_x, scale_y = 2,2

# Crear una imagen escalada en blanco
scaled_img = np.zeros((int(x * scale_y), int(y * scale_x)), dtype=np.uint8)

# Escalar la imagen
for i in range(int(x * scale_y)):
    for j in range(int(y * scale_x)):
        orig_x = int(i / scale_y)
        orig_y = int(j / scale_x)
        if 0 <= orig_x < x and 0 <= orig_y < y:
            scaled_img[i, j] = img[orig_x, orig_y]

# Aplicar convolución manual con kernel 3x3 de valores 1/9
convoluted_img = np.zeros_like(scaled_img, dtype=np.uint8)
kernel = [[1/9, 1/9, 1/9], [1/9, 1/9, 1/9], [1/9, 1/9, 1/9]]
kernel_size = 3
offset = kernel_size // 2

# Convolución
for i in range(offset, scaled_img.shape[0] - offset):
    for j in range(offset, scaled_img.shape[1] - offset):
        sum_val = 0
        for ki in range(kernel_size):
            for kj in range(kernel_size):
                pixel_val = scaled_img[i + ki - offset, j + kj - offset]
                sum_val += pixel_val * kernel[ki][kj]
        convoluted_img[i, j] = np.clip(sum_val, 0, 255)

# Mostrar las imágenes
cv.imshow('Imagen Original', img)
cv.imshow('Imagen Escalada', scaled_img)
cv.imshow('Imagen Convolucionada (Suavizado)', convoluted_img)
cv.waitKey(0)
cv.destroyAllWindows()
