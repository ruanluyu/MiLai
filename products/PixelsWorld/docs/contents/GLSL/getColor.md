#getColor
---
This function has two versions. **getColor(vec2)** and **getColor(int ,vec2)**.
##getColor(vec2)
**Description : **
 - Get the color of **inLayer** in specified position. 

**Parameters : **
- vec2 : uv position.

**Return : **
- vec4 : a RGBA color.

>**NOTE : **
> We highly recommend using ```getColor(uv);``` to obtain pixels, we cannot guarantee that ```texture2d(gl_Position,sampler2d);``` can return the right pixel. 

##getColor(int,vec2)
**Description : **
 - Get the color of **specified layer** in specified position. 

**Parameters : **
- int : layer id, -1 means inLayer, number between 0 and 9 means the corresponding Layer Parameter in your plugin panel. 
- vec2 : uv coordinates.

**Return : **
- vec4 : a RGBA color. 

>**NOTE : **
> - So in other words, ```getColor(-1,uv);``` totally equals to ```getColor(uv);```
> - ```getColor(n,uv);``` can return the pixels of layer[n]. 