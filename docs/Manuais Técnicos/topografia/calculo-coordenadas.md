# Cálculo de Coordenadas Topográficas

Este manual documenta o processo básico para o cálculo de coordenadas retangulares (E, N) a partir de uma poligonal topográfica, utilizando a distância horizontal e o azimute.

## 1. Fórmulas Fundamentais

Para encontrar as coordenadas do ponto de vante (B) a partir de um ponto de estação (A) conhecido, utilizamos as seguintes equações trigonométricas:

$$E_B = E_A + D_{AB} \cdot \sin(Az_{AB})$$

$$N_B = N_A + D_{AB} \cdot \cos(Az_{AB})$$

Onde:
* **E, N:** Coordenadas Este e Norte.
* **D:** Distância horizontal entre os pontos.
* **Az:** Azimute da alinhamento.

!!! warning "Atenção aos Quadrantes"
    Ao realizar os cálculos em planilhas ou scripts, lembre-se que os azimutes variam de 0° a 360°. As funções de seno e cosseno nas linguagens de programação geralmente exigem que o ângulo seja convertido de graus decimais para radianos antes do cálculo.

---

## 2. Automação: Calculando e Exportando para CAD

Em vez de calcular ponto a ponto na calculadora, podemos usar um script simples em Python para processar uma lista de distâncias e azimutes e já formatar a saída para importar os pontos no ZWCAD ou AutoCAD.

```python
import math

def calcular_coordenada(e_origem, n_origem, distancia, azimute_graus):
    # Converte o azimute para radianos
    azimute_rad = math.radians(azimute_graus)
    
    # Aplica as fórmulas trigonométricas
    e_destino = e_origem + (distancia * math.sin(azimute_rad))
    n_destino = n_origem + (distancia * math.cos(azimute_rad))
    
    return round(e_destino, 3), round(n_destino, 3)

# Exemplo de uso:
estacao_E, estacao_N = 1000.000, 2000.000
dist = 45.50
az = 125.30 # Graus decimais

novo_E, novo_N = calcular_coordenada(estacao_E, estacao_N, dist, az)

print(f"Comando CAD: POINT {novo_E},{novo_N}")