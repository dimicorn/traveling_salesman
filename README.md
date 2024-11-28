# Научная задача
## Условие
**Даны линии, по которым проходит разведка. В данных пара точек задает линию. Нужно реализовать алгоритм, который обойти все линии за минимальное евклидовое расстояние.**

*По сути своей перед нами стоит задача о поиске гамильтонова пути в полной неориентированном графе с неотрицательными весами, или метрическая задача о коммивояжере (Traveling salesman problem). Эта задача NP-полная, поэтому с решениями все достаточно грустно.*

## Точное решение
Существует, конечно же, метод полного перебора, но у него временная сложность $O(n!)$. Для $n = 300$, $n! = 300! \sim 3 \cdot 10^{614}$ операций или примерно $10^{599}$ лет вычислений. По самым большим оценкам возраст Вселенной составляет $15 \cdot 10^8$ лет, поэтому было решено сразу перейти к более "оптимальному" алгоритму.

В 1962 году, Хелдом и Карпом был разработан [алгоритм](https://jurnalinternasional.wordpress.com/wp-content/uploads/2011/01/dpsequencing.pdf) для нахождения точного решения с асимптотической сложностью $\Theta(2^n n^2)$ по времени и $\Theta(n2^n)$ дополнительной памяти.

Пронумеруем города $1, 2, \dots, n$, причем $1$ может быть произвольно выбран в качестве "стартового" города (поскольку решением задачи является гамильтонов цикл, выбор стартового города не имеет значения). Алгоритм Хелда-Карпа начинает с вычисления для каждого набора городов $S \subseteq \{2, \dots, n\}$ и каждого города $e \neq 1$, не содержащегося в $S$, кратчайшего одностороннего пути из $1$ в $e$, который проходит через все города в $S$ в некотором порядке (но не через любые другие города). Обозначим это расстояние $g(S, e)$ и напишем $d(u, v)$ для длины прямого ребра из $u$ в $v$. Мы будем вычислять значения $g(S, e)$, начиная с наименьших множеств $S$ и заканчивая наибольшими.

Предположим, что $S=\{s_1,\dots,s_k\}$ - это множество из $k$ городов. Для каждого целого числа $1 \leq i \leq k$ запишем $S_i=S\setminus\{s_i\}=\{s_1,\dots,s_{i-1},s_{i+1},\dots,s_k\}$ для множества, образованного удалением $s_i$ из $S$. Тогда если кратчайший путь из $1$ через $S$ в $e$ имеет $s_i$ в качестве предпоследнего города, то удаление последнего ребра из этого пути должно дать кратчайший путь из $1$ в $s_i$ через $S_i$. Это означает, что существует только $k$ возможных кратчайших путей из $1$ в $e$ через $S$, по одному для каждого возможного предпоследнего города $s_i$ с длиной $g(S_i,s_i)+d(s_i,e)$, а $g(S,e)=\min_{1\leq i\leq k} g(S_i,s_i)+d(s_i,e)$.
Этот этап алгоритма завершается, когда для каждого целого числа $2\leq i\leq n$ известно $g(\{2,\dots,i-1,i+1,\dots,n\},i)$, дающее кратчайшее расстояние от города $1$ до города $i$, которое проходит через все остальные города. На гораздо более коротком втором этапе эти расстояния добавляются к длинам ребер $d(i,1)$, чтобы получить $n-1$ возможных кратчайших циклов, и затем находится кратчайший.

Наконец, сам кратчайший путь (а не только его длина) может быть восстановлен путем хранения рядом с $g(S,e)$ метки предпоследнего города на пути из $1$ в $e$ через $S$, что увеличивает требования к пространству лишь на постоянный коэффициент.

****
## Приближенное решение
В 1993 году Дориго описал [метод](https://www.sciencedirect.com/science/article/pii/S0303264797017085) эвристической генерации "хороших решений" задачи коммивояжера с помощью симуляции муравьиной колонии, названной ACS (ant colony system). Она моделирует поведение, наблюдаемое у реальных муравьев для поиска коротких путей между источниками пищи и их гнездом - поведение, являющееся результатом предпочтения каждого муравья следовать феромонам, отложенным другими муравьями.

ACS посылает большое количество виртуальных агентов-муравьев для исследования множества возможных маршрутов на карте. Каждый муравей вероятностно выбирает следующий город для посещения, основываясь на эвристике, сочетающей расстояние до города и количество виртуального феромона, отложенного на пути к городу:
$$w = \frac{P_{ij}^\alpha}{D_{ij}^\beta}, \text{где }P \text{ - количество феромонов на пути, а }D \text{ - длина пути}.$$
Муравьи исследуют город, оставляя феромон на каждом краю, который они проходят, пока все они не завершат тур. В этот момент муравей, совершивший самый короткий тур, оставляет виртуальный феромон по всему маршруту своего тура (глобальное обновление маршрутов). Количество отложенного феромона обратно пропорционально длине пути: чем короче путь, тем больше откладывается феромона.

Временная сложность такого алгоритма $O(n^2 mt)$, где $t$ - количество запусков поиска оптимального маршрута, а $m$ - количество муравьев в колонии.

## Результаты
*Для всех запусков ACS были зафиксированы* $m = 50$, $\alpha = 0.9$, $\beta = 1.5$.
### $25$ городов
|<!-- -->|Held-Karp|ACS $t = 100$|ACS $t = 1000$|
|---|---|---|---|
|Длина пути|426363.62|646756.40|521197.86|
|Путь|0->8->10->12->14->5->3->16->20->22->24->21->17->7->1->18->19->23->15->13->11->9->6->4->2|19->15->23->7->1->21->17->24->22->20->18->12->14->16->3->5->8->10->11->9->0->4->2->6->13|15->19->23->5->3->16->20->22->24->21->17->7->1->18->14->12->10->8->11->9->0->2->4->6->13|
|Время работы|133.646 сек|0.670431 сек|6.86016 сек|

### $300$ городов
Для $n = 300$ точное решение начинает падать с ошибкой `Segmentation fault`, вероятно код пытается выделить слишком много дополнительной памяти для решения задачи.

|<!-- -->|ACS $t = 100$|ACS $t = 1000$|
|---|---|---|
|Длина пути|1277131.92|1241397.20|
|Время работы|43.8598 сек|439.16 сек|
<!-- 69 37 263 55 102 203 136 222 133 298 293 106 177 287 235 71 116 206 234 166 35 78 256 231 49 233 66 161 98 153 296 68 156 26 40 6 214 165 31 123 50 290 252 193 169 80 198 112 75 43 237 23 285 10 217 152 85 186 212 201 199 33 240 254 276 105 149 48 221 250 192 2 179 121 215 113 87 295 286 65 73 46 185 288 12 11 64 142 19 96 95 282 239 280 15 175 16 264 289 151 129 197 230 273 13 223 261 101 184 39 172 176 47 74 200 76 54 220 242 44 141 83 299 249 258 42 63 1 67 125 32 36 114 180 24 77 137 53 241 190 196 0 294 118 218 269 209 274 143 14 187 150 131 154 202 4 104 38 130 260 132 111 182 248 255 208 108 72 97 86 100 189 226 245 159 135 119 145 270 253 225 228 56 107 167 243 219 88 5 89 236 265 162 120 155 284 148 124 160 34 45 246 41 139 110 259 51 18 9 291 277 3 146 144 126 157 251 195 90 27 205 268 62 99 158 147 170 278 17 25 174 204 232 91 138 79 140 163 292 279 281 82 210 272 173 207 213 262 247 188 227 20 92 122 29 8 267 61 52 81 58 297 266 216 224 28 178 30 117 238 57 275 211 171 7 22 257 271 94 244 60 229 70 109 59 127 84 164 283 194 103 93 134 181 128 183 168 21 191 115  -->

<!-- 187 29 277 68 156 218 236 265 141 294 276 107 196 241 243 71 116 206 234 167 14 143 252 210 73 272 79 140 135 147 297 43 98 17 53 15 151 289 21 121 58 291 247 193 166 109 180 24 111 33 182 59 287 7 231 235 63 172 183 195 176 45 250 221 188 85 152 49 256 269 254 0 127 70 229 132 88 293 296 114 94 30 244 285 10 76 50 174 153 145 178 283 194 279 13 168 19 142 288 64 106 177 215 251 11 157 126 26 137 77 120 162 56 22 200 74 48 160 191 34 131 89 290 204 238 35 78 1 67 133 46 41 139 205 39 69 179 28 208 122 115 2 282 92 212 267 159 245 219 72 190 161 155 217 224 27 184 9 128 263 90 108 214 202 130 123 61 52 81 119 20 227 38 104 4 124 149 100 189 253 266 99 57 249 197 237 171 101 5 95 230 165 154 209 223 284 274 118 60 25 40 268 62 103 117 175 18 47 16 280 298 3 169 148 80 198 203 144 146 84 216 270 54 65 220 93 211 233 23 42 275 258 299 125 201 32 36 75 295 271 260 96 226 242 105 225 228 261 281 170 278 112 102 248 55 8 262 186 87 44 91 292 246 207 199 83 164 129 158 264 31 232 192 181 12 51 259 163 136 255 37 273 113 97 185 213 6 138 286 222 110 82 239 173 86 134 66 240 257 150 -->

## Запуск кода
Для запуска программы с гиперпараметрами по умолчанию $\alpha = 0.9$, $\beta = 1.5$, $m = 50$, $t = 100$ необходимо дать права на запуск скрипту:
```console
$ chmod +x run.sh
```
и запустить скрипт передав ему файл с координатами городов:
```console
$ ./run.sh input.txt
```
