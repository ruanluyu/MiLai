# テンプレート聖域shadertoy
---
## 使用方法
このモードでは、[shadertoy.com](https://shadertoy.com) のコードを直接使用することができます。

![shadertoyMode](shadertoyMode.png)

> **注意** <br/>shadertoy.comからのすべてのコードがPixelsWorldで完全に動作するわけではありません (例: オーディオ入力が必要なもの、VRサポートが必要なもの、ループバッファ参照を含むコードなど)

![loopRef](loopbuffer.png)

## 詳細
shadertoyモードで独自にパラメータパネルを呼び出したい場合は、GLSLモード下のすべての変数に**`_PixelsWorld_`**プレフィックスを付けることで利用可能です。

> 例えば、GLSLモード下の`slider[0]`は`_PixelsWorld_slider[0]`になります。

すべてのグローバル変数の定義については、[こちら](./predefined.md#shadertoy-mode)を参照してください。

もちろん、shadertoyモードで**`_PixelsWorld_`**プレフィックスを持つ変数を独自に定義することはお勧めしません。

## バッファーの使用

Shadertoyには4つのバッファーがあり、PixelsWorldでもバッファーを実装することができます。
### ループしないバッファー

![luaMode](../Lua/LuaMode.png)

使用しているエフェクトに、バッファー間のループ呼び出しがない場合（上図の下半分のようなもの）は、エフェクト内のバッファーはループしないバッファーに属します。それらを呼び出すには、以下のコードテンプレートを使用します（注意：Luaモードに切り替える必要があります）。

例：https://www.shadertoy.com/view/4dVGRW からのコード

1. このエフェクトは`BufferA`を使用しているので、以下のコードの4行目で`A`を`true`に設定します。
2. `BufferA`は最終的に`Main`画像の`iChannel0`にバインドされるので、以下のコードの15行目で`AtoMain`を`0`に設定します。
3. その後、`BufferA`と`Main`のコードをそれぞれ対応する変数にコピーするだけです。
4. 最後に、計算結果が正確になるように、プラグインパネルの`Advanced->Internal texture format`を`Floating point 32 bit x RGBA (HDR)`に設定して、PixelsWorldが内部でアルファチャンネルの値を削らないようにします。
5. Luaレンダーモードであることを確認してください。

```lua:rotatingCubes.lua
version3()

enable = {
    A=true,
    B=false,
    C=false,
    D=false,

    -- '-1': 無効化
    -- '0~3': どのiChannelに設定するか
    
    AtoB = -1,
    AtoC = -1,
    AtoD = -1,
    AtoMain = 0,

    BtoC = -1,
    BtoD = -1,
    BtoMain = -1,

    CtoD = -1,
    CtoMain = -1,

    DtoMain = -1,
}

mainCode = [==[
// ここにメインコードを貼り付けます。
mat3 calcLookAtMatrix(vec3 origin, vec3 target, float roll) {
    vec3 rr = vec3(sin(roll), cos(roll), 0.0);
    vec3 ww = normalize(target - origin);
    vec3 uu = normalize(cross(ww, rr));
    vec3 vv = normalize(cross(uu, ww));
  
    return mat3(uu, vv, ww);
  }
  
  vec3 getRay(vec3 origin, vec3 target, vec2 screenPos, float lensLength) {
    mat3 camMat = calcLookAtMatrix(origin, target, 0.0);
    return normalize(camMat * vec3(screenPos, lensLength));
  }
  
  vec2 squareFrame(vec2 screenSize, vec2 coord) {
    vec2 position = 2.0 * (coord.xy / screenSize.xy) - 1.0;
    position.x *= screenSize.x / screenSize.y;
    return position;
  }
  
  vec2 getDeltas(sampler2D buffer, vec2 uv) {
    vec2 pixel = vec2(1. / iResolution.xy);
    vec3 pole = vec3(-1, 0, +1);
    float dpos = 0.0;
    float dnor = 0.0;
      
    vec4 s0 = texture(iChannel0, uv + pixel.xy * pole.xx); // x1, y1
    vec4 s1 = texture(iChannel0, uv + pixel.xy * pole.yx); // x2, y1
    vec4 s2 = texture(iChannel0, uv + pixel.xy * pole.zx); // x3, y1
    vec4 s3 = texture(iChannel0, uv + pixel.xy * pole.xy); // x1, y2
    vec4 s4 = texture(iChannel0, uv + pixel.xy * pole.yy); // x2, y2
    vec4 s5 = texture(iChannel0, uv + pixel.xy * pole.zy); // x3, y2
    vec4 s6 = texture(iChannel0, uv + pixel.xy * pole.xz); // x1, y3
    vec4 s7 = texture(iChannel0, uv + pixel.xy * pole.yz); // x2, y3
    vec4 s8 = texture(iChannel0, uv + pixel.xy * pole.zz); // x3, y3
  
    dpos = (
      abs(s1.a - s7.a) +
      abs(s5.a - s3.a) +
      abs(s0.a - s8.a) +
      abs(s2.a - s6.a)
    ) * 0.5;
    dpos += (
      max(0.0, 1.0 - dot(s1.rgb, s7.rgb)) +
      max(0.0, 1.0 - dot(s5.rgb, s3.rgb)) +
      max(0.0, 1.0 - dot(s0.rgb, s8.rgb)) +
      max(0.0, 1.0 - dot(s2.rgb, s6.rgb))
    );
    
    dpos = pow(max(dpos - 0.5, 0.0), 5.0);
      
    return vec2(dpos, dnor);
  }
  
  void mainImage(out vec4 fragColor, in vec2 fragCoord) {
    vec3 ro = vec3(sin(iTime * 0.2), 1.5, cos(iTime * 0.2)) * 5.;
    vec3 ta = vec3(0, 0, 0);
    vec3 rd = getRay(ro, ta, squareFrame(iResolution.xy, fragCoord.xy), 2.0);
    vec2 uv = fragCoord.xy / iResolution.xy;
      
    vec4 buf = texture(iChannel0, fragCoord.xy / iResolution.xy);
    float t = buf.a;
    vec3 nor = buf.rgb;
    vec3 pos = ro + rd * t;
      
    vec3 col = vec3(0.5, 0.8, 1);
    vec2 deltas = getDeltas(iChannel0, uv);
    if (t > -0.5) {
      col = vec3(1.0);
      col *= max(0.3, 0.3 + dot(nor, normalize(vec3(0, 1, 0.5))));
      col *= vec3(1, 0.8, 0.35);
    }
    col.r = smoothstep(0.1, 1.0, col.r);
    col.g = smoothstep(0.1, 1.1, col.g);
    col.b = smoothstep(-0.1, 1.0, col.b);
    col = pow(col, vec3(1.1));
    col -= deltas.x - deltas.y;
    
      
    fragColor = vec4(col, 1);
  }
]==];

commonCode = [==[
// ここに共通コードを貼り付けます。

]==]

bufferACode = [==[
// ここにBuffer Aのコードを貼り付けます。
float sdBox( vec3 p, vec3 b ) {
    vec3 d = abs(p) - b;
    return min(max(d.x,max(d.y,d.z)),0.0) +
           length(max(d,0.0));
  }
  
  vec2 mirror(vec2 p, float v) {
    float hv = v * 0.5;
    vec2  fl = mod(floor(p / v + 0.5), 2.0) * 2.0 - 1.0;
    vec2  mp = mod(p + hv, v) - hv;
      
    return fl * mp;
  }
  
  vec2 rotate2D(vec2 p, float a) {
    return p * mat2(cos(a), -sin(a), sin(a),  cos(a));
  }
  
  float map(vec3 p) {
    float r = iMouse.z > 0.0 ? iMouse.x / 100.0 : iTime * 0.9;
    p.xz = mirror(p.xz, 4.);
    p.xz = rotate2D(p.xz, r);
    float d = sdBox(p, vec3(1));
    d = min(d, sdBox(p, vec3(0.1, 0.1, 3)));
    d = min(d, sdBox(p, vec3(3, 0.1, 0.1)));
    return d;
  }
  
  mat3 calcLookAtMatrix(vec3 origin, vec3 target, float roll) {
    vec3 rr = vec3(sin(roll), cos(roll), 0.0);
    vec3 ww = normalize(target - origin);
    vec3 uu = normalize(cross(ww, rr));
    vec3 vv = normalize(cross(uu, ww));
  
    return mat3(uu, vv, ww);
  }
  
  vec3 getRay(vec3 origin, vec3 target, vec2 screenPos, float lensLength) {
    mat3 camMat = calcLookAtMatrix(origin, target, 0.0);
    return normalize(camMat * vec3(screenPos, lensLength));
  }
  
  float calcRayIntersection(vec3 rayOrigin, vec3 rayDir, float maxd, float precis) {
    float latest = precis * 2.0;
    float dist   = +0.0;
    float type   = -1.0;
    float res    = -1.0;
  
    for (int i = 0; i < 30; i++) {
      if (latest < precis || dist > maxd) break;
  
      float result = map(rayOrigin + rayDir * dist);
  
      latest = result;
      dist  += latest;
    }
  
    if (dist < maxd) {
      res = dist;
    }
  
    return res;
  }
  
  vec2 squareFrame(vec2 screenSize, vec2 coord) {
    vec2 position = 2.0 * (coord.xy / screenSize.xy) - 1.0;
    position.x *= screenSize.x / screenSize.y;
    return position;
  }
  
  vec3 calcNormal(vec3 pos, float eps) {
    const vec3 v1 = vec3( 1.0,-1.0,-1.0);
    const vec3 v2 = vec3(-1.0,-1.0, 1.0);
    const vec3 v3 = vec3(-1.0, 1.0,-1.0);
    const vec3 v4 = vec3( 1.0, 1.0, 1.0);
  
    return normalize( v1 * map( pos + v1*eps ) +
                      v2 * map( pos + v2*eps ) +
                      v3 * map( pos + v3*eps ) +
                      v4 * map( pos + v4*eps ) );
  }
  
  vec3 calcNormal(vec3 pos) {
    return calcNormal(pos, 0.002);
  }
  
  void mainImage(out vec4 fragColor, in vec2 fragCoord) {
    vec2 uv = squareFrame(iResolution.xy, fragCoord.xy);
    vec3 ro = vec3(sin(iTime * 0.2), 1.5, cos(iTime * 0.2)) * 5.;
    vec3 ta = vec3(0, 0, 0);
    vec3 rd = getRay(ro, ta, uv, 2.0);
      
    float t = calcRayIntersection(ro, rd, 20.0, 0.001);
    vec3 pos = ro + rd * t;
    vec3 nor = calcNormal(pos);
      
    fragColor = vec4(nor, t);
  }
]==];

bufferBCode = [==[
// ここにBuffer Bのコードを貼り付けます。

]==];

bufferCCode = [==[
// ここにBuffer Cのコードを貼り付けます。

]==];

bufferDCode = [==[
// ここにBuffer Dのコードを貼り付けます。

]==];


-- == 警告 警告 警告 警告 警告 == --
-- == この行より下のコードは修正しないでください。 == --

-- コード By ZzStarSound 
-- このプリセットコードはPixelsWorld内でのみ使用してください。 
-- コードの乱用は禁止されています。 


swapTexDet = function(sch,tch)
    if sch == nil or tch == nil then return end
    if sch >= 0 and sch <= 3 then
        swapTex(sch,tch)
    end
end


if enable.A then 
    texA = newTex(width,height)
    shadertoy(commonCode .. bufferACode)
    swapTex(texA,OUTPUT)
end

if enable.B then
    texB = newTex(width,height)
    swapTexDet(enable.AtoB,texA)
    shadertoy(commonCode .. bufferBCode)
    swapTexDet(enable.AtoB,texA)
    swapTex(texB,OUTPUT)
end

if enable.C then
    texC = newTex(width,height)
    swapTexDet(enable.AtoC,texA)
    swapTexDet(enable.BtoC,texB)
    shadertoy(commonCode .. bufferCCode)
    swapTexDet(enable.AtoC,texA)
    swapTexDet(enable.BtoC,texB)
    swapTex(texC,OUTPUT)
end

if enable.D then
    texD = newTex(width,height)
    swapTexDet(enable.AtoD,texA)
    swapTexDet(enable.BtoD,texB)
    swapTexDet(enable.CtoD,texC)
    shadertoy(commonCode .. bufferDCode)
    swapTexDet(enable.AtoD,texA)
    swapTexDet(enable.BtoD,texB)
    swapTexDet(enable.CtoD,texC)
    swapTex(texD,OUTPUT)
end

swapTexDet(enable.AtoMain,texA)
swapTexDet(enable.BtoMain,texB)
swapTexDet(enable.CtoMain,texC)
swapTexDet(enable.DtoMain,texD)
shadertoy(commonCode .. mainCode)
swapTexDet(enable.AtoMain,texA)
swapTexDet(enable.BtoMain,texB)
swapTexDet(enable.CtoMain,texC)
swapTexDet(enable.DtoMain,texD)
```
### ループ バッファー

うーん...もしループバッファーを使用したい場合、PixelsWorldでそれらを使用することは強くお勧めしません。（しかし、実現することは可能です。AeとPixelsWorldとの長い夜の戦いに備える必要があります。[準備ができましたか>>>](./../Lua/Simulation.md)）