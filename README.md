# Localización y Clasificación de Personajes de Los Simpson mediante Redes Neuronales Convolucionales y Modelos Multi-Output

Este repositorio contiene el desarrollo completo del Trabajo Práctico Integral para la materia **Introducción a Visión por Computadora** de la **Licenciatura en Ciencia de Datos (UNSAM)**[cite: 1, 2]. El proyecto aborda el desafío de clasificar e identificar espacialmente (mediante *Bounding Boxes*) a los personajes de la serie "Los Simpson" en capturas de pantalla de los episodios, utilizando técnicas de Procesamiento Digital de Imágenes (PDI) y Deep Learning[cite: 1, 2].

---

## Dataset y Desafíos Visuales

Se utilizó un subconjunto balanceado del **The Simpsons Characters Dataset** (Alex Attia, Kaggle), bajo licencia CC BY-NC-SA 4.0[cite: 1, 2]. 

### Desafíos del Dominio:
* **Proximidad Cromática:** Todos los personajes comparten el mismo tono de piel amarillo[cite: 2]. El modelo no puede depender del color, lo que lo fuerza a aprender estructuras geométricas complejas (formas de pelo, ojos, accesorios)[cite: 2].
* **Fondos Complejos:** Escenas con alto ruido visual (la taberna de Moe, la planta nuclear, etc.) que confunden a los extractores de características tradicionales[cite: 2].
* **Variabilidad de Poses:** Personajes de frente, perfil, tres cuartos y con deformaciones propias de la animación[cite: 2].

---

## Pipeline de Desarrollo

### 1. Preprocesamiento y Balanceo de Datos
* **Mitigación del Sesgo:** El dataset original presentaba un severo desbalanceo (clases con 2000 imágenes y otras con 400)[cite: 2]. Se seleccionaron 6 clases principales (`Homer`, `Bart`, `Lisa`, `Marge`, `Ned Flanders`, `Montgomery Burns`) y se limitó el volumen a **300 imágenes por personaje**, consolidando un subset perfectamente balanceado de **1800 imágenes**[cite: 1, 2].
* **Normalización:** Redimensionamiento uniforme a $128 \times 128 \times 3$ y normalización de píxeles en el rango $[0, 1]$[cite: 1].
* **Partición Estratificada:** División 80% Train y 20% Test garantizando la equidad de clases en ambos conjuntos mediante `stratify`[cite: 2].

### 2. Análisis de Procesamiento Digital de Imágenes (PDI)
Antes del modelado por Deep Learning, se evaluaron técnicas clásicas de PDI[cite: 2]:
* **Ecualización de Histograma:** Incrementó el contraste en escenas oscuras pero introdujo ruido y saturación en altas luces, destruyendo geometría útil para las redes[cite: 2].
* **Algoritmo de Canny (Bordes):** Aisló contornos eficientemente pero demostró ser altamente sensible al ruido y eliminó la información de color de los bloques de animación[cite: 2].
* **Conclusión:** Se determinó que el formato nativo RGB es superior para el entrenamiento de arquitecturas convolucionales en este dominio[cite: 2].

---

## Arquitecturas de Modelado y Experimentos

### Experimento 1: Red Totalmente Conectada (ANN)
Se aplanaron las imágenes a vectores unidimensionales[cite: 2]. 
* **Resultado:** Rendimiento nulo (Accuracy estancado en **16.67%**, equivalente al azar)[cite: 2]. Al destruir la topología bidimensional y la relación de vecindad entre píxeles, la red sufrió de *underfitting* severo y sesgo absoluto hacia la clase mayoritaria en las iteraciones iniciales[cite: 2].

### Experimento 2: CNN propia vs. Transfer Learning
* **CNN desde Cero:** Se diseñó una red con 3 bloques `Conv2D` + `MaxPooling2D`[cite: 1]. Alcanzó un ~71% de accuracy en validación, pero sufrió de un *overfitting* temprano debido al volumen acotado de datos[cite: 2].
* **Transfer Learning (MobileNetV2):** Utilizando la base convolucional congelada de `MobileNetV2` (preentrenada en ImageNet), el modelo estabilizó su accuracy de validación en **75%** con curvas de pérdida notablemente más convergentes, demostrando que los filtros abstractos preentrenados mitigan el ruido de los fondos cargados[cite: 2].

### Experimento 3: Modelo Multi-Output (Clasificación + Regresión)
Se diseñó una arquitectura funcional capaz de resolver dos tareas en simultáneo mediante una bifurcación tras el *Global Average Pooling* de `MobileNetV2`[cite: 2]:
1. **Rama de Clasificación:** Salida Softmax (6 neuronas) con pérdida *Sparse Categorical Crossentropy*[cite: 2].
2. **Rama de Regresión (Bounding Boxes):** Salida Sigmoide (4 neuronas) con pérdida *Mean Squared Error (MSE)* para predecir las coordenadas espaciales normalizadas[cite: 2].

> **Función de Pérdida Ponderada:** 
> $$\mathcal{L}_{total} = 1.0 \times \mathcal{L}_{clase} + 2.0 \times \mathcal{L}_{caja}$$
> Se penalizó con el doble de peso el error de localización debido a la complejidad de converger coordenadas continuas desde cero[cite: 2].

---

## Resultados y Métricas Finales

* **Localización (MSE):** Excelente convergencia, estabilizando el MSE de validación en **0.018**, demostrando alta precisión en el encuadre de los personajes[cite: 2].
* **Clasificación (Accuracy):** Se estabilizó entre el **73% y 77%** en validación[cite: 2]. La ligera baja respecto al modelo puro se debe a la competencia por los recursos algorítmicos en las capas convolucionales compartidas al optimizar dos tareas a la vez[cite: 2].

---

## Créditos e Informe 
Proyecto desarrollado por **Oscar Mauricio Kreszczuk** para la Licenciatura en Ciencia de Datos, UNSAM[cite: 1, 2]. 
El informe teórico y metodológico completo con las conclusiones detalladas se encuentra adjunto en este repositorio como `Informe_vision.pdf`[cite: 1].
