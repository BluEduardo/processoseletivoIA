# Relatório do Candidato - Desafio Edge AI

👤 **Identificação:** Eduardo Roberto Pereira

## 1️⃣ Resumo da Arquitetura do Modelo
O modelo implementado no ficheiro `train_model.py` é uma Rede Neural Convolucional (CNN) desenhada estritamente sob o paradigma de **Edge AI**. Em vez de procurar a máxima complexidade, a arquitetura prioriza a eficiência e o baixo consumo de recursos (RAM e Flash):

* **Entrada:** Imagens de 28x28 píxeis com 1 canal (tons de cinzento).
* **Extração de Características:** Utiliza apenas duas camadas convolucionais (`Conv2D`) com um número intencionalmente reduzido de filtros (8 e 16), alternadas com camadas `MaxPooling2D` para redução da dimensionalidade espacial. Esta escolha reduz drasticamente a quantidade de parâmetros matemáticos necessários.
* **Regularização:** Uma camada `Dropout` com taxa de 0.3 para evitar o *overfitting*, garantindo a generalização sem adicionar qualquer custo computacional na fase de inferência.
* **Classificação:** Uma camada `Flatten` seguida de uma camada densa super compacta (32 neurónios) e a camada de saída `softmax` (10 classes).

## 2️⃣ Bibliotecas Utilizadas
Para garantir a máxima estabilidade no pipeline de integração contínua (CI) e respeitar o ambiente delimitado, foram utilizadas exclusivamente:
* **TensorFlow / Keras (>=2.12):** Utilizada como *framework* principal para a construção da arquitetura, treino do modelo em CPU e posterior conversão.
* **NumPy:** Utilizada para o pré-processamento dimensional dos tensores e para o cálculo algébrico seguro de métricas avançadas (F1-Score).

## 3️⃣ Técnica de Otimização do Modelo
No ficheiro `optimize_model.py`, apliquei a técnica de **Dynamic Range Quantization** (Quantização de Intervalo Dinâmico) através do conversor do TensorFlow Lite (`tf.lite.Optimize.DEFAULT`).

Esta técnica converte estaticamente os pesos e parâmetros do modelo, passando-os de números de ponto flutuante de 32 bits (*float32*) para números inteiros de 8 bits (*int8*). O *trade-off* assumido foi a troca de uma fração impercetível de precisão decimal por uma redução colossal do tamanho do modelo. Esta otimização permite que o modelo caiba na reduzida memória de microcontroladores (como a SRAM de um ESP32) e utilize as instruções de números inteiros do processador para inferências muito mais rápidas.

## 4️⃣ Resultados Obtidos
* **Treino (Desempenho):** O modelo atingiu convergência em apenas 5 épocas, garantindo uma **Acurácia superior a 98%** nos dados de teste e um **F1-Score Macro de ~0.98**, o que prova que a rede generalizou bem para todos os dígitos sem enviesamentos.
* **Otimização (Tamanho):** A quantização provou a sua eficácia ao reduzir o modelo original de **~209 KB** (formato `.h5`) para apenas **~20 KB** (formato `.tflite`), alcançando uma taxa de compressão de cerca de **90%**.

## 5️⃣ Comentários Adicionais

* **Decisões Técnicas Importantes (Segurança no CI/CD):** Para cumprir a exigência de utilizar "mais de uma métrica" sem quebrar o *pipeline* de correção, optei por calcular o *F1-Score* de forma manual utilizando apenas laços de repetição e NumPy. Evitei importar bibliotecas externas como o `scikit-learn`, o que poderia originar um erro fatal (ModuleNotFoundError) no GitHub Actions caso não constasse no `requirements.txt` da infraestrutura.
* **Aprendizagens e Trade-offs:** O maior desafio técnico foi encontrar o equilíbrio (*trade-off*) perfeito entre precisão e tamanho. Foi fundamental compreender que, no contexto de sistemas embarcados, um modelo de 3 MB com 99.9% de acurácia é inútil se causar um *Overflow* na memória RAM do microcontrolador. A solução de 20 KB entregue sacrifica menos de 1% de precisão teórica, mas viabiliza o *deploy* real num dispositivo Edge da vida real.
