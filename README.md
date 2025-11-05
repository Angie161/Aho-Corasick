# Algoritmo Aho-Corasick  
**Angie Verónica Ramírez González**  
*Matrícula: 2022453214*

---

## 1. Convenciones y Notación

A lo largo de este informe se emplean las siguientes notaciones:

- **m**: suma total de las longitudes de todos los patrones de la lista.  
- **n**: longitud del texto donde se buscan los patrones.  
- **z**: número total de ocurrencias encontradas durante la búsqueda.  
- **k**: tamaño del alfabeto empleado.

---

## 2. Definición

**Aho-Corasick** es un algoritmo diseñado para la búsqueda simultánea de múltiples patrones dentro de un texto. Construye un autómata finito basado en un *trie* en tiempo **O(m)** y permite procesar el texto en una única lectura, notificando todas las apariciones de los patrones en tiempo **O(n + z)**.

A grandes rasgos, el algoritmo Aho-Corasick se compone de las siguientes etapas:

### 1 Recepción de los patrones  
Se recibe la lista con todos los patrones que deben buscarse dentro del texto.

### 2 Construcción del *trie*  
A partir de la lista de patrones, se construye un *trie* para representar todas las secuencias de caracteres de forma compacta.

Para esto se comienza desde la raíz e inserta cada patrón carácter por carácter.  
- Si no existe una arista asociada al carácter actual, se crea un nuevo nodo y se le conecta al *trie* a través de una arista etiquetada con dicho carácter. Más adelante, estas conexiones serán conocidas como **transiciones**.
- Por otra parte, si la arista ya existe, se continúa a través de ella y se repite la secuencia hasta completar el patrón.

> **Figura 1.** Ejemplo de construcción de un *trie* con conjunto de patrones {morsa, orca}.

Durante la construcción, cada nodo del *trie* posee información relevante para el funcionamiento posterior del autómata.  
Para efectos de mejor comprensión, se puede asociar a cada nodo una etiqueta con la cadena formada desde la raíz hasta ese punto, facilitando la visualización del proceso.

Respecto a los aspectos técnicos, cada nodo debe contener:

- Una **flag** denominada output, que indica si el nodo corresponde al final de alguno de los patrones buscados.  
- Un **arreglo o mapa de transiciones**, que especifica a qué nodos se puede acceder dependiendo del carácter leído.  
  - Si se implementa mediante un arreglo, este deberá ser de tamaño `k`, teniendo un uso de memoria **O(mk)**.  
  - Si se emplea un `map`, el uso de memoria se reduce a **O(m)** a costa de un tiempo de acceso **O(log k)**.  
- Un **suffix link** y un **arreglo de output links**, que se explicarán más adelante.

---

### 3. Creación del Autómata de Estado Finito

A partir de la estructura del *trie* y sus aristas de transición, el algoritmo adquiere una forma similar a un **autómata finito**, donde cada nodo representa un estado y cada arista una transición etiquetada por un carácter. Sin embargo, para que sea completamente determinista, debe existir una transición definida para cada carácter del alfabeto.

En principio no se quiere construir todas las posibles transiciones desde cada nodo a todos los demás con cada elemento del alfabeto, ya que eso sería extremadamente costoso, por lo que el algoritmo introduce un mecanismo alternativo: los **suffix links**, que permiten “completar” implícitamente las transiciones faltantes sin necesidad de definirlas todas.

Notar que en los *tries*, cada cadena almacenada es un prefijo de alguna otra, por lo que, cuando se está en un nodo *u* y se quiere transicionar con un carácter *c* que no posee una transición directa, se busca el **sufijo propio más largo** del string representado por *u* que se encuentre en el *trie*.  
Desde ese nuevo nodo se intenta procesar nuevamente el carácter *c*, generando un comportamiento similar a poseer todas las transiciones definidas (Figura 2). 

La idea detrás de esta construcción es optimizar la búsqueda durante la lectura del texto. A medida que se procesa el texto y se desciende por el *trie*, puede ocurrir que el carácter actual no tenga una transición válida en la rama que se está revisando. En lugar de reiniciar la búsqueda desde la raíz, el algoritmo utiliza los **suffix links** para crear “atajos” que permitan continuar la exploración sin perder el contexto acumulado. Estos atajos redirigen la búsqueda hacia otro patrón de la lista que podría coincidir con la parte del texto que se está procesando. Gracias a este mecanismo, se aprovecha al máximo la información obtenida hasta ese punto y se evita repetir comparaciones innecesarias, logrando una búsqueda más eficiente y continua.

> **Figura 2.** Ejemplo de funcionamiento de los *suffix links* en un *trie* construido con los patrones {morsa, orca}.

En el ejemplo de la Figura 2, se está procesando el texto “morca”. Dado que al llegar al nodo *u* no existen transiciones con el carácter “c”, se redirige la búsqueda al sufijo propio más largo de “mor”, llegando hasta “or”. Así, aunque inicialmente se haya intentado reconocer el patrón “morsa” de forma equívoca, se logra encontrar el patrón “orca” sin reiniciar la búsqueda.

---

### 3.1 Creación de Suffix Links

Para implementar este mecanismo, se deberá recorrer el *trie* con una **búsqueda en anchura (BFS)**, y por cada nodo se definirá el siguiente algoritmo:

I. El nodo raíz y todos sus hijos directos tendrán un `suffix_link` que apunta hacia la raíz.  
II. Para cualquier otro nodo `v`: Sea `u` el nodo padre de `v`, y `c` el carácter en la arista `(u, v)`. Desde el nodo `u`, seguir su `suffix_link` hacia un nodo `w`. Si `w` posee una transición con `c`, el `suffix_link` de `v` apuntará al nodo alcanzado por dicha transición. En caso contrario, repetir el seguimiento de los `suffix_links` hasta encontrar una transición válida o llegar a la raíz.

> **Figura 3.** Ejemplo de construcción de `suffix_link` en un *trie* construido con los patrones {morsa, orca}.

---

### 3.2 Creación de Output Links

Durante el procesamiento del texto, pueden encontrarse patrones adicionales que no coinciden directamente con la rama actual del *trie* (por ejemplo, cuando un patrón es prefijo de otro). Para manejar estos casos, cada nodo mantiene un **output link** que permite reconocer dichos patrones implícitos.

En concreto, si un nodo `v` tiene un `suffix_link` que apunta a un nodo terminal (es decir, que representa un patrón completo), el `output_link` de `v` apuntará hacia ese nodo. De esta forma, al recorrer `v`, también se detectan los patrones asociados a sus `output_links`.

Dentro del alrgoritmo, estos enlaces se generan en paralelo a los `suffix_links`: si un `suffix_link` apunta a un nodo terminal, dicho enlace también se considera un `output_link`.

---

## 4. Ejemplo de Uso

Sea `T` el *trie* construido a partir de los patrones: {"sal","al","mal","ma","a"} (Figura 4). 

> Figura 4: **Trie** construido a partir de los patrones `{sal, al, mala, a, ma}`, con sus correspondientes **suffix links** en azul, **output links** en rojo y nodos enumerados.

Se busca encontrar las coincidencias de estos patrones en el texto **“salamandra”**.  
Para efectos de simplificación y claridad en el proceso, se definirán las siguientes funciones asociadas al autómata:

- \( f(i, c) \): función de transición desde el nodo *i* mediante el carácter *c*. Devuelve el nodo al que se transiciona.  
- \( s(i, c) \): función que retorna el nodo alcanzado al seguir el *suffix link* del nodo *i* y aplicar el carácter *c*.  
- \( o(i) \): función que entrega el conjunto de patrones asociados al nodo *i* a través de sus *output links*.

La siguiente tabla resume las transacciones realizadas en cada carácter del texto, junto con la cantidad de coincidencias de los patrones que se hayan encontrado en cada paso.

| Paso | Char | Estado inicial |     Transición     | Estado final | ¿Estado terminal? |  o(v)  | Patrones detectados |
|:----:|:----:|:---------------:|:------------|:---------------:|:-----------------:|:----:|:--------------------|
| 1 | s | 0 | \( f(0, s) = 1 \) | 1 | No | – | – |
| 2 | a | 1 | \( f(1, a) = 2 \) | 2 | No | \( o(2) = 4 \) | [“a”] |
| 3 | l | 2 | \( f(2, l) = 3 \) | 3 | Sí | \( o(3) = 5 \) | [“sal”, “al”] |
| 4 | a | 3 | \( f(3, a) = s(3, a) = s(5, a) = f(0, a) = 4 \) | 4 | Sí | – | [“a”] |
| 5 | m | 4 | \( f(4, m) = s(4, m) = f(0, m) = 6 \) | 6 | No | – | – |
| 6 | a | 6 | \( f(6, a) = 7 \) | 7 | Sí | \( o(7) = 4 \) | [“ma”, “a”] |
| 7 | n | 7 | \( f(7, n) = s(7, n) = f(0, n) = 0 \) | 0 | No | – | – |
| 8 | d | 0 | \( f(0, d) = 0 \) | 0 | No | – | – |
| 9 | r | 0 | \( f(0, r) = 0 \) | 0 | No | – | – |
| 10 | a | 0 | \( f(0, a) = 4 \) | 4 | Sí | – | [“a”] |

---

### Notas aclaratorias del proceso

- **Paso 4:**  
  \( f(3, a) = s(3, a) = s(5, a) = f(0, a) = 4 \)  
  No existe transición desde el nodo 3 con el carácter “a”, por lo que se sigue su *suffix link* hacia el nodo 5.  
  Dado que desde allí tampoco existe dicha transición, se sigue el *suffix link* de 5, llegando a la raíz y realizando la transición con “a” hacia el nodo 4.

- **Paso 7:**  
  \( f(7, n) = s(7, n) = f(0, n) = 0 \)  
  Si no existe transición desde la raíz, se retorna la misma raíz.

Por último, una taba más compacta de las transiciones y coincidencias de patrones; si hay una marca en la tabla significa que existe coincidencia de uno o más patrones:
| Texto | | s | | a | | l | | a | | m | | a | | n | | d | | r | | a | |
|:------|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| Estado | 0 | → | 1 | → | 2 | → | 3 | → | 4 | → | 6 | → | 7 | → | 0 | → | 0 | → | 0 | → | 4 |
| Coincidencia | | | | | ✓ | | ✓ | | ✓ | | | | ✓ | | | | | | | | ✓ |

---

## 5. Implementación

A continuación se muestra la implementación del algoritmo Aho-Corasick desarrollada en C++.

```cpp
#include <bits/stdc++.h>
using namespace std;

struct Node {
    unordered_map<char,int> next;   //char de la transición y el número del siguiente estado.
    int suffix_link = -1;
    vector<int> output_links;      // El vector de output links también registrará si se es un nodo terminal.
    Node() = default;        
};

class AhoCorasick {
public:
    vector<Node> trie;
    vector<string> patterns;

    AhoCorasick(vector<string> p) {
        patterns = p; 
        trie.emplace_back();

        for (int i = 0; i < (int)patterns.size(); ++i){ 
            insert(patterns[i], i);
        }
        build_FSM();
    } 

    // Inserta un patrón y guarda su id (índice en patterns)
    void insert(const string &patt, int id) {
        int v = 0;
        for (char c : patt) {
            auto it = trie[v].next.find(c);
            // Si no se encuentra en el trie, añadir un estado
            if (it == trie[v].next.end()) {
                trie[v].next[c] = (int)trie.size();
                trie.emplace_back();
                v = (int)trie.size() - 1;
            } else {
                v = it->second;
            }
        }
        trie[v].output_links.push_back(id);
    }

    // Construye el Autómata de estado finito, agregando suffix links (Con BFS) y output links.
    void build_FSM() {
        trie[0].suffix_link = 0;
        queue<int> q;
        
        // Inicializar hijos directos de la raíz
        for (auto &dc : trie[0].next) {
            int child = dc.second;
            trie[child].suffix_link = 0;
            q.push(child);
        }

        // BFS para la construcción
        while (!q.empty()) {
            int v = q.front(); q.pop();

            for (auto &u : trie[v].next) {
                char ch_u = u.first;
                int id_u = u.second;

                q.push(id_u);

                // hallar suffix para u: seguir suffix_link(s) desde suffix_link(v) buscando ch_u
                int j = trie[v].suffix_link;
                while (j != 0 && trie[j].next.find(ch_u) == trie[j].next.end()) {
                    j = trie[j].suffix_link;
                }
                if (trie[j].next.find(ch_u) != trie[j].next.end())
                    trie[id_u].suffix_link = trie[j].next[ch_u];
                else
                    trie[id_u].suffix_link = 0;

                // agregar output_links
                for (int pid : trie[trie[id_u].suffix_link].output_links)
                    trie[id_u].output_links.push_back(pid);

            }
        }

        // Solo por si acaso para remover posibles duplicados en los output_links
        for (auto &node : trie) {
            sort(node.output_links.begin(), node.output_links.end());
            node.output_links.erase(unique(node.output_links.begin(), node.output_links.end()), node.output_links.end());
        }
    }

    // Busca en texto y devuelve vector de (pattern_id, end_pos)
    // end_pos es el índice en text donde termina la coincidencia
    vector<pair<int,int>> search(const string &text) {
        vector<pair<int,int>> matches;
        int v = 0;
        for (int i = 0; i < (int)text.size(); ++i) {
            char ch = text[i];
            // seguir transiciones; si no hay, seguir suffix links
            while (v != 0 && trie[v].next.find(ch) == trie[v].next.end()) {
                v = trie[v].suffix_link;
            }
            if (trie[v].next.find(ch) != trie[v].next.end()) {
                v = trie[v].next.at(ch);
            } else {
                v = 0;
            }

            // reportar todas las salidas en este estado
            for (int pid : trie[v].output_links) {
                matches.emplace_back(pid, i);
            }
        }
        return matches;
    }

    // Utilidad: número de nodos en el trie
    size_t size() const { return trie.size(); }
};
```
## 6. Referencias

- Aho, A. V., & Corasick, M. J. (1975). *Efficient string matching: An aid to bibliographic search*. *Communications of the ACM*. Recuperado de [https://cr.yp.to/bib/1975/aho.pdf](https://cr.yp.to/bib/1975/aho.pdf)

- cp-algorithms. (2025). *Aho-Corasick algorithm*. *Algorithms for Competitive Programming*. Recuperado de [https://cp-algorithms.com/string/aho_corasick.html](https://cp-algorithms.com/string/aho_corasick.html)

- Stanford University. (s. f.). *Aho-Corasick Automata (CS166 — Lecture slides)*. Recuperado de [https://web.stanford.edu/class/archive/cs/cs166/cs166.1166/lectures/02/Slides02.pdf](https://web.stanford.edu/class/archive/cs/cs166/cs166.1166/lectures/02/Slides02.pdf)

---

## 7. Material Complementario

- ComputerBread. (2024, julio 20). *Ctrl+F on steroids - Aho-Corasick Algorithm (pt. 1)* [Video]. YouTube.  
  [https://www.youtube.com/watch?v=XWujo7KQL54](https://www.youtube.com/watch?v=XWujo7KQL54)

- ComputerBread. (2024, julio 27). *Aho-Corasick Algorithm - JavaScript Implementation (pt. 2)* [Video]. YouTube.  
  [https://www.youtube.com/watch?v=jsgLCvOW6Vo](https://www.youtube.com/watch?v=jsgLCvOW6Vo)
