[[vectors#addition]]
## additions
Exercise 1: Let `a = [2, 3]` and `b = [-1, 4]`. Compute `a + b` by hand.
 solution: $$
\begin{aligned}
(x^1 + x^2, y^1 + y^2) &= (2 + (-1), 3 + 4) \\
                       &= (1, 7)
\end{aligned}
$$
Exercise 2: Let `a = [1, 0, 5]` and `b = [2, -3, 1]`. Compute `a + b` and `a - b`
 solution:
 ``` 
 a=[1,0,5], b=[2,-3,1]
 a+b =[1+2],[0+(-3)],[5+1]                   a-b = [1-2],[0-(-3)],[5-1]
 
 a+b = [3,-3,6]                              a-b = [-1,3,4]
	 
 ```
 
 Exercise 3: Prove to yourself that `a + b = b + a` using your vectors from Exercise 1.
```
a = [2, 3]` and `b = [-1, 4]
	a+b= [2 + (-1)],[3+4]         b+a = [-1 + 2],[4+3]
	a+b= [1,7]                    b+a = [1,7]
	a+b,b+a=[1,7] 
	hence proved => a+b = b+a
```

Exercise 4: A drone is at position `[0, 0]`. It moves `[3, 4]` km north-east, then `[-1, 2]` km. Where does it end up?

```
a=[3,4],b=[-1,2]
a+b=[3+(-1)],[4+2]
a+b = [2,6]
```
which means that in the end the drone's positions was `[2,6]`

## Magnitude Practice

[[vectors#Magnitude (Length of Vector)]]

Exercise 5: Compute `|a|` for `a = [3, 4]`. (Hint: 3-4-5 triangle.)

$$|a| = \sqrt{3^2+4^2} = \sqrt{9+16} = \sqrt{25} = 5 $$
          => |a| = 5

Exercise 6: Compute `|v|` for `v = [1, 1, 1]`.
$$|v| =\sqrt{1^2+1^2+1^2} = \sqrt{1+1+1} =\sqrt{3} = 1.73 $$


Exercise 7: What is the magnitude of the zero vector `[0, 0, 0]`?
$$|v| = \sqrt{0^2+0^2+0^2} =\sqrt{0} = 0$$

since magnitute means length of the vector so if both axis are at origin point then the lengh of that vector will be nothing since it never began


Exercise 8: Find a vector in 2D with magnitude exactly 1. (This is called a unit vector.)
$$if -\vec{v} = [0,1] then, |v| = \sqrt{0^2+1^2}=\sqrt{0+1}=\sqrt{1}=1 $$
it should be right since the vector is travelingin only one direction it has to be its length


I'm skipping dot product I'll do it tomorrow 

## dot 
fds
