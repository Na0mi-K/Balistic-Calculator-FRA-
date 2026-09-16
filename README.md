# Programme par NaomiK , FluoroPolymers™

***Calculer des trajectoires balistiques sans utiliser aucune trigonométrie:***

**Les équations différentielles séparées avec une masse m, une gravité g, un coefficient de frottement k (posons $\alpha = k/m$) et des vitesses initiales $v_{x0}, v_{y0}$ sont :**

$$\ddot{x} + \alpha \dot{x} = 0$$
$$\ddot{y} + \alpha \dot{y} = -g$$

**Les solutions exactes pour les positions en fonction du temps t s'écrivent :**

$$x(t) = \frac{v_{x0}}{\alpha} \left(1 - e^{-\alpha t}\right)$$
$$y(t) = \frac{1}{\alpha} \left(v_{y0} + \frac{g}{\alpha}\right) \left(1 - e^{-\alpha t}\right) - \frac{g}{\alpha} t$$

**Développement en série entière (Série de Taylor)En utilisant le développement en série entière usuel de $e^{-X}$ :**

$$e^{-\alpha t} = \sum_{n=0}^{\infty} \frac{(-\alpha t)^n}{n!} = 1 - \alpha t + \frac{\alpha^2 t^2}{2!} - \frac{\alpha^3 t^3}{3!} + \dots$$

**On obtient directement les séries entières polynomiales pour $x(t)$ et $y(t)$ :**

$$x(t) = v_{x0} \sum_{n=1}^{\infty} \frac{(-1)^{n-1} \alpha^{n-1}}{n!} t^n = v_{x0} t - \frac{v_{x0} \alpha}{2} t^2 + \frac{v_{x0} \alpha^2}{6} t^3 - \frac{v_{x0} \alpha^3}{24} t^4 + \dots$$
$$y(t) = v_{y0} t - \left(g + \alpha v_{y0}\right) \frac{t^2}{2} + \frac{\alpha g + \alpha^2 v_{y0}}{6} t^3 - \frac{\alpha^2 g + \alpha^3 v_{y0}}{24} t^4 + \dots$$

**Approximation polynomiale d'ordre 4Pour une approximation très précise à petit/moyen terme sans calcul d'exponentielle ni de trigonométrie :**

$$x(t) \approx v_{x0} t \left(1 - \frac{\alpha t}{2} + \frac{\alpha^2 t^2}{6} - \frac{\alpha^3 t^3}{24}\right)$$
$$y(t) \approx v_{y0} t - \frac{g + \alpha v_{y0}}{2} t^2 + \frac{\alpha(g + \alpha v_{y0})}{6} t^3 - \frac{\alpha^2(g + \alpha v_{y0})}{24} t^4$$

**Calcul sans trigonométrie pour l'angle d'attaque**
**Si la vitesse initiale V0 et l'angle theta sont donnés sous forme de vecteur $(u_x, u_y)$ unitaire au lieu d'un angle en degrés :**
- $v_{x0} = V_0 \cdot u_x$
- $v_{y0} = V_0 \cdot u_y$
Cela évite totalement l'usage de cos(theta) et sin(theta).

**On peut la mettre sous une forme encore plus “compacte” en factorisant par $(g+αv_y0)$ :**

$$y(t)\approx v_\text{y0}t - \frac{(g+\alpha v_\text{y0})}{2}t^2 + \frac{\alpha(g+\alpha v_\text{y0})}{6}t^3 - \frac{\alpha ^2(g+\alpha v_\text{y0})}{24}t^4$$

**Ou, si on veut un schéma de Horner pour l’évaluation numérique .**

$$x(t) \approx v_{x_0}t \left( 1 + t \left( -\frac{\alpha}{2} + t \left( \frac{\alpha^2}{6} + t \left( -\frac{\alpha^3}{24} \right) \right) \right) \right)$$

$$\boxed{y(t) \approx t \left[ v_{y_0} + t \left( -\frac{g + \alpha v_{y_0}}{2} + t \left( \frac{\alpha(g + \alpha v_{y_0})}{6} + t \left( -\frac{\alpha^2(g + \alpha v_{y_0})}{24} \right) \right) \right) \right]}$$

Cela minimise le nombre d’opérations et évite toute fonction transcendante.

**La série converge pour tout t (exponentielle entière), mais l’approximation tronquée à l’ordre 4 n’est précise que tant que 𝛼𝑡 reste “modéré”. Un ordre de grandeur utile :**
- Si $αt≲0.5$ , l’erreur relative sur $x(t)$ et $y(t)$ est typiquement de l’ordre de αt^5/120 donc très petite.
- Si $αt≳1$ , la troncature à l’ordre 4 commence à montrer des écarts notables par rapport à la solution exponentielle exacte.

En pratique, pour des projectiles dans l’air avec résistance linéaire (modèle idéalisé, valable plutôt à faible nombre de Reynolds), α est souvent petit, donc sur la durée typique de vol, αt peut rester dans un domaine où l’ordre 4 est excellent
- *On peut Augmenter l’ordre du développement (5, 6, …) jusqu’à ce que le terme suivant soit négligeable*
- *Utiliser la forme exponentielle exacte (souvent très bien optimisée sur CPU/GPU).*

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

ça marche pour des projectiles très légers mais une vraie résistance de l'air n'est pas linéaire . *Si mon projectile a une masse de 1 Kg ou plus il va falloir calculer les choses quadratiquement et ça change tout car c'est plus compliqué en termes de solutions formelles.* J'ai quelques options devant moi : 

- Un tir purement horizontal (*c'est nul et trop limité*)
- Développement en série de ***Taylor quadratique*** : calcul des dérivées successives à t=0 pour réobtenir un polynôme à évaluer en Horner, bien que sa portée temporelle soit plus limitée qu'en linéaire

ou alors la solution vraiment intéressante à mon sens : L' Intégration numérique explicite (Euler semi-implicite / Verlet)

- **RK4** — plus précis à pas égal, un peu plus de code, utile si vous voulez comparer la précision numérique elle-même (au-delà de la comparaison Taylor vs exact qu'on a déjà déjà).

j'ai choisi ***Euler*** car Verlet demande une structure de code que je n'aime pas , et franchement Euler était bien plus simple à intégrer à mon code sans perte de précision . Par exemple pour un projectile :
- de 5 Kg 
- tiré à 60° 
- à 650 m/s 

Le programme actuel prédit ***exactement*** le comportement d'une telle situation .

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

 # Point Important : 

 **Ici le modèle prend en compte des objets quelconques sans propriétés aérodynamiques avantageuses. e coefficient de frottement choisi représente donc un air relativement dense ou un projectile très peu aérodynamique (un obus réel de $5\text{ kg}$ aurait un $k_2$ nettement plus faible et une bien meilleure portée).**
 
 *( La simulation est correcte et réaliste pour un objet subissant une forte traînée aérodynamique (* $k_2 = 0,01$ *)*
 

------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
<img width="1300" height="550" alt="Image" src="https://github.com/user-attachments/assets/173cdba8-6e5b-4d9e-a2fd-13c985286e7c" />
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------




