# Vectors

A **vector** is an arrow in 2D or 3D space.

- In **2D** → $(x, y)$ → moves along x (sideways) and y (up/down)  
- In **3D** → $(x, y, z)$ → adds depth (z direction)

📌 Each value tells how far the arrow goes in that direction.

![[Screenshot 2026-03-21 214838.png]]

---

## Magnitude (Length of Vector)

Size (length) of a vector:

[[exercises#dot]]

$$
|\vec{v}| = \sqrt{x^2 + y^2}
$$

Example: $(3,4)$  
$$
|\vec{v}| = \sqrt{3^2 + 4^2} = 5
$$

👉 Same idea as **Pythagoras theorem**

---

## Operations

## addition

[[exercises#Addition Practic]]

- **2D:** $(x_1,y_1)+(x_2,y_2)=(x_1+x_2,\ y_1+y_2)$  
- **3D:** $(x_1,y_1,z_1)+(x_2,y_2,z_2)=(x_1+x_2,\ y_1+y_2,\ z_1+z_2)$  

---

## Scaling

[[symbols and formulas#scaling]]
Multiply a vector by a number (scalar).

- $k>1$ → stretch  
- $0<k<1$ → shrink  
- $k=0$ → zero vector  
- $k<0$ → flip direction  

Examples:  
```
2(3,4)=(6,8)
-1(3,4)=(-3,-4)
```
**Magnitude rule:**  

$$ |k\vec{v}| = |k|\cdot|\vec{v}| $$


Example: $(3,4)\to5,\quad 2\vec{v}\to10$
