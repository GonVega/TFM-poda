# Documentación: Métodos de Optimización de Post-Poda

## Índice
1. [Introducción](#introducción)
2. [Post-Poda Clásica](#post-poda-clásica)
3. [Poda Greedy para Velocidad](#poda-greedy-para-velocidad)
4. [Comparación de Métodos](#comparación-de-métodos)
5. [Exportación a If-Else](#exportación-a-if-else)
6. [Ejemplos de Uso](#ejemplos-de-uso)
7. [Recomendaciones](#recomendaciones)

---

## Introducción

Este proyecto implementa dos métodos de optimización de post-poda para árboles de decisión generados a partir de redes neuronales. Los métodos permiten reducir la complejidad del árbol manteniendo la precisión del modelo.

### Objetivos de la Poda
- **Reducir el número de nodos** del árbol
- **Mantener la precisión** del modelo
- **Acelerar la inferencia** 
- **Mejorar la interpretabilidad**

---

## Post-Poda Clásica

### Descripción
La post-poda clásica recorre el árbol en orden postorden y evalúa si cada nodo puede ser convertido en hoja sin degradar significativamente el rendimiento del modelo.

### Algoritmo
1. **Recorrido postorden**: Se visita cada nodo después de sus hijos
2. **Evaluación de poda**: Para cada nodo:
   - Se convierte temporalmente en hoja
   - Se asigna la clase mayoritaria de las muestras que llegan al nodo
   - Se evalúa el rendimiento en el conjunto de validación
3. **Decisión de poda**: 
   - Si el rendimiento no empeora → se mantiene la poda
   - Si el rendimiento empeora → se restaura el nodo original

### Implementación
```python
from nndt_lib.post_pruning import post_prune_tree

# Aplicar post-poda clásica
post_prune_tree(tree, X_val, y_val, task='classification', metric='accuracy')
```

### Parámetros
- `tree`: Árbol de decisión a podar
- `X_val, y_val`: Conjunto de validación
- `task`: 'classification' o 'regression'
- `metric`: Métrica a optimizar ('accuracy', 'mean_squared_error', etc.)

### Ventajas
- ✅ Conservadora: mantiene la precisión
- ✅ Robusta: funciona bien en diferentes datasets
- ✅ Interpretable: fácil de entender

### Desventajas
- ❌ Puede no reducir mucho el árbol si el modelo está bien ajustado
- ❌ Depende de la calidad del conjunto de validación

---

## Poda Greedy para Velocidad

### Descripción
La poda greedy es más agresiva que la clásica. Permite una pequeña pérdida de precisión (controlada por tolerancia) a cambio de una mayor reducción del árbol.

### Algoritmo
1. **Recorrido postorden**: Similar a la post-poda clásica
2. **Evaluación con tolerancia**: Para cada nodo:
   - Se convierte temporalmente en hoja
   - Se evalúa el rendimiento
   - Se compara con el rendimiento original
3. **Decisión de poda**:
   - Si la pérdida ≤ tolerancia → se acepta la poda
   - Si la pérdida > tolerancia → se rechaza la poda

### Implementación
```python
from nndt_lib.prune_for_speed import prune_tree_for_speed

# Aplicar poda greedy con tolerancia baja
prune_tree_for_speed(tree, X_val, y_val, 
                    task='classification', 
                    metric='accuracy', 
                    tolerance=0.00005)
```

### Parámetros
- `tree`: Árbol de decisión a podar
- `X_val, y_val`: Conjunto de validación
- `task`: 'classification' o 'regression'
- `metric`: Métrica a optimizar
- `tolerance`: Tolerancia máxima de pérdida de rendimiento

### Ventajas
- ✅ Más agresiva: reduce más el árbol
- ✅ Controlable: parámetro de tolerancia ajustable
- ✅ Eficiente: mayor reducción de tiempo de inferencia

### Desventajas
- ❌ Puede degradar la precisión si la tolerancia es muy alta
- ❌ Requiere ajuste del parámetro de tolerancia
- ❌ Menos robusta que la post-poda clásica

---

## Comparación de Métodos

| Aspecto | Post-Poda Clásica | Poda Greedy |
|---------|-------------------|-------------|
| **Conservadurismo** | Alta | Baja |
| **Reducción de nodos** | Moderada | Alta |
| **Mantenimiento de precisión** | Excelente | Variable |
| **Facilidad de uso** | Alta | Media |
| **Ajuste de parámetros** | Mínimo | Requerido |
| **Robustez** | Alta | Media |

### Ejemplo de Resultados Típicos

```
Árbol Original:
- Nodos: 1023
- Accuracy: 0.7482
- Tiempo: 1.14s

Post-Poda Clásica:
- Nodos: 229
- Accuracy: 0.7514
- Tiempo: 1.14s

Poda Greedy (tolerance=0.00005):
- Nodos: 171
- Accuracy: 0.7418
- Tiempo: 1.14s
```

---

## Exportación a If-Else

### Descripción
Para máxima eficiencia en inferencia, el árbol podado puede exportarse a una función Python en formato if-else puro, eliminando la sobrecarga de objetos.

### Implementación
```python
from nndt_lib.tree_to_ifelse import export_tree_to_ifelse

# Exportar árbol a función if-else
code = export_tree_to_ifelse(tree)
with open('tree_ifelse_exported.py', 'w') as f:
    f.write(code)

# Usar la función generada
local_vars = {}
exec(code, globals(), local_vars)
tree_predict = local_vars['tree_predict']

# Inferencia ultrarrápida
predictions = [tree_predict(x) for x in X_test]
```

### Ventajas de la Exportación
- ⚡ **Inferencia ultrarrápida**: Sin sobrecarga de objetos
- 📦 **Portabilidad**: Código Python puro
- 🔧 **Simplicidad**: Fácil de integrar en otros sistemas
- 📊 **Consistencia**: Reproduce exactamente el comportamiento del árbol

---

## Ejemplos de Uso

### Ejemplo 1: Post-Poda Clásica
```python
from nndt_lib.binary_tree import Tree
from nndt_lib.post_pruning import post_prune_tree

# Crear árbol desde red neuronal
tree = Tree(model)
tree.create_DT(auto_prune=True, verbose=False)

# Aplicar post-poda clásica
post_prune_tree(tree, X_val, y_val, task='classification', metric='accuracy')

# Evaluar resultados
acc = tree.evaluate_model(X_test, y_test, task='classification', metrics=['accuracy'])
print(f"Accuracy tras post-poda: {acc['accuracy']:.4f}")
```

### Ejemplo 2: Poda Greedy
```python
from nndt_lib.prune_for_speed import prune_tree_for_speed

# Aplicar poda greedy con tolerancia baja
prune_tree_for_speed(tree, X_val, y_val, 
                    task='classification', 
                    metric='accuracy', 
                    tolerance=0.00005)

# Evaluar resultados
acc = tree.evaluate_model(X_test, y_test, task='classification', metrics=['accuracy'])
print(f"Accuracy tras poda greedy: {acc['accuracy']:.4f}")
```

### Ejemplo 3: Combinación de Métodos
```python
# 1. Post-poda clásica
post_prune_tree(tree, X_val, y_val, task='classification', metric='accuracy')

# 2. Poda greedy (opcional)
prune_tree_for_speed(tree, X_val, y_val, 
                    task='classification', 
                    metric='accuracy', 
                    tolerance=0.00005)

# 3. Exportar a if-else
code = export_tree_to_ifelse(tree)
with open('final_tree.py', 'w') as f:
    f.write(code)
```

---

## Recomendaciones

### Cuándo Usar Post-Poda Clásica
- ✅ Cuando la precisión es crítica
- ✅ En producción donde la robustez es importante
- ✅ Cuando no se requiere máxima reducción del árbol
- ✅ Para datasets pequeños o medianos

### Cuándo Usar Poda Greedy
- ✅ Cuando se necesita máxima reducción del árbol
- ✅ Cuando se puede tolerar una pequeña pérdida de precisión
- ✅ Para sistemas con restricciones de memoria/tiempo
- ✅ Cuando se tiene un buen conjunto de validación

### Ajuste de Parámetros

#### Tolerancia para Poda Greedy
```python
# Muy conservadora (mantiene precisión)
tolerance = 0.00001

# Moderada (balance entre precisión y reducción)
tolerance = 0.00005

# Agresiva (máxima reducción)
tolerance = 0.001
```

### Flujo Recomendado
1. **Entrenar red neuronal** y convertir a árbol
2. **Aplicar post-poda clásica** como base
3. **Evaluar rendimiento** en conjunto de validación
4. **Opcionalmente aplicar poda greedy** con tolerancia baja
5. **Exportar a if-else** para inferencia ultrarrápida

### Monitoreo
- 📊 Comparar accuracy antes y después de cada poda
- ⏱️ Medir tiempo de inferencia
- 📈 Contar número de nodos y profundidad media
- 🔍 Verificar que las predicciones sean consistentes

---

## Conclusión

Los dos métodos de post-poda ofrecen diferentes estrategias de optimización:

- **Post-poda clásica**: Conservadora, robusta, ideal para producción
- **Poda greedy**: Agresiva, eficiente, ideal para sistemas con restricciones

La combinación de ambos métodos, seguida de la exportación a if-else, proporciona la máxima optimización manteniendo la precisión del modelo.

La elección entre métodos dependerá de los requisitos específicos de cada aplicación: precisión vs. eficiencia, robustez vs. simplicidad.

---

## Anexo: Descripción Académica para TFM

### Post-Poda Clásica

La post-poda clásica implementa un algoritmo de optimización basado en el principio de minimización del error de generalización. Este método recorre el árbol de decisión en orden postorden, evaluando cada nodo interno para determinar si puede ser convertido en un nodo hoja sin degradar significativamente el rendimiento del modelo. El proceso se fundamenta en la hipótesis de que ciertos subárboles pueden ser redundantes o contribuir al sobreajuste del modelo, especialmente cuando se han generado a partir de redes neuronales complejas.

La estrategia de evaluación utiliza un conjunto de validación independiente para medir el impacto de cada operación de poda. Para cada nodo candidato, el algoritmo simula temporalmente su conversión a hoja asignándole la clase mayoritaria de las muestras que llegan a ese nodo. Si la métrica de rendimiento (típicamente accuracy para clasificación) no se deteriora tras esta modificación, la poda se mantiene; en caso contrario, se restaura la estructura original del nodo. Este enfoque conservador garantiza que la precisión del modelo se mantenga o mejore, aunque puede resultar en una reducción más moderada de la complejidad del árbol.

### Poda Greedy para Velocidad

La poda greedy para velocidad representa una extensión más agresiva del algoritmo de post-poda clásica, introduciendo un parámetro de tolerancia que permite un control explícito sobre el trade-off entre simplicidad del modelo y precisión. A diferencia del método conservador anterior, esta técnica acepta una degradación controlada del rendimiento a cambio de una reducción más sustancial en la complejidad computacional del árbol. El algoritmo mantiene la estructura de recorrido postorden pero modifica el criterio de decisión para incorporar una tolerancia predefinida.

La innovación principal de este método radica en su capacidad de balancear automáticamente la precisión y la eficiencia mediante el parámetro de tolerancia. Este valor actúa como un umbral que determina la máxima pérdida de rendimiento aceptable para cada operación de poda individual. El proceso iterativo evalúa cada nodo candidato comparando la métrica de rendimiento antes y después de la simulación de poda, aceptando la modificación únicamente si la diferencia no excede el umbral establecido. Esta aproximación permite alcanzar reducciones significativas en el número de nodos y la profundidad media del árbol, lo que se traduce en mejoras sustanciales en los tiempos de inferencia, especialmente críticas en aplicaciones que requieren latencia mínima o procesamiento en tiempo real. 