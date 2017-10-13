---
categories: Physics
...

*Snell's law* is a formula used to describe the relationship between the angles of incidence and refraction, when referring to light or other waves passing through a boundary between two different isotropic media, such as water, glass, or air.

It is also of importance in some [optimization](Mathematical optimization) problems. For example, a lifeguard running to save a drowning person in the ocean will take a path that follows Snell's law.
If the lifeguard can run at speed $v_1$ on the beach, and swim at speed $v_2$ in the ocean, then the angle $\theta_1$ at which the lifeguard enters the ocean (the angle of incidence) and the angle $\theta_2$ at which the lifeguard swims towards the drowning person (the angle of refraction) satisfies Snell's law if he take's the shortest possible path:
$$ \frac{\sin\theta_1}{v_1} = \frac{\sin\theta_2}{v_2}. $$

> **Insight**: If Snell's law gives an undefined result (perhaps because of a domain error for $\sin^{-1}$), that means the light wouldn't be refracted, but reflected instead. In terms of optimization, this means that the given scenario is not optimal.

## Problems
- [EllysThreeRivers](https://community.topcoder.com/stat?c=problem_statement&pm=11911&rd=14735) [^1]
- [RemoteRover](https://community.topcoder.com/stat?c=problem_statement&pm=4022&rd=6534) [^2]
- [Ironman](https://open.kattis.com/problems/ironman)

## External links
- [Snell's law](https://en.wikipedia.org/wiki/Snell%27s_law)


[^1]: <https://apps.topcoder.com/wiki/display/tc/SRM+543>
[^2]: <https://www.topcoder.com/tc?module=Static&d1=match_editorials&d2=srm235>
