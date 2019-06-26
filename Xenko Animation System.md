# Overview of Animations

One way to view the essence of an game engine, is to treat it as **a long-lasting interactive piece of animation**. Everything players see, everything players do, is in essence **interacting and viewing animations**. This simplified view is not considering **UI** and other **information presentation** mediums or **simulation based interactions**, but it does allows us to think of **user interaction** with the game in terms of *a sequence of animations happening in a predefined order*.

With that in mind, there are several ways to represent animations in an efficient manner:

1. **Time based scripting:** this is like *procedural animation* of object parameters depending on time; **User interaction** can easily be incorporated in this process
2. **(3D) Skeletal Animation Clips**
3. **(2D) Sprite Animations**
4. **(2D&3D) Particle Animations**

We shall also notice, the largest containing unit for such animations is the **scene** itself - by switching scenes we can essentially present different animations in a discrete manner.


See:

1. https://doc.xenko.com/latest/jp/manual/animation/animation-properties.html
2. (Control custom attributes with a script)[https://doc.xenko.com/latest/en/manual/animation/custom-attributes.html#2-control-custom-attributes-with-a-script]
3. https://doc.xenko.com/latest/jp/manual/animation/animation-scripts.html
2. Also see animation related Code Snippets