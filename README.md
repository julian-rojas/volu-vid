## aframe-volu-vid-component

Volumetric Video component for [A-Frame](https://aframe.io) , created with DepthKit in Mind.

### Properties

| Property              | Description               | Default Values    |
|-----------------------|---------------------------|-------------------|
| name                  | filename w/o ext          | video             |
| depthFocalLengthX     | Unknown DepthKit Var*      | 365.2185974121094 |
| depthFocalLengthY     | Unknown DepthKit Var*      | 365.2185974121094 |
| depthImageSizeX       | Full Width of Webm Frame  | 512               |
| depthImageSizeY       | Half Height of Webm Frame | 424               |
| depthPrincipalPointX  | Unknown DepthKit Var*      | 257.6430969238281 |
| depthPrincipalPointY  | Unknown DepthKit Var*      | 209.3957977294922 |
| farClip               | Unknown DepthKit Var*      | 1204.192016601562 |
| nearClip              | Unknown DepthKit Var*      | 434.166748046875  |

*These should be assigned based on the _meta.txt file created by DepthKit

### Usage

```html
<head>
  <title>My A-Frame Scene</title>
  <script src="https://aframe.io/releases/0.7.0/aframe.min.js"></script>
  <script src="https://unpkg.com/aframe-randomizer-components@^3.0.1/dist/aframe-randomizer-components.min.js"></script>
</head>

<!-- The following two shaders must be included. It seems easiest to leave them in the HTML for now -->
    <script id="vs" type="x-shader/x-vertex">
      vec3 rgb2hsl( vec3 color ) {
        float h = 0.0;
        float s = 0.0;
        float l = 0.0;
        float r = color.r;
        float g = color.g;
        float b = color.b;
        float cMin = min( r, min( g, b ) );
        float cMax = max( r, max( g, b ) );
        l =  ( cMax + cMin ) / 2.0;
        if ( cMax > cMin ) {
          float cDelta = cMax - cMin;
          // saturation
          if ( l < 0.5 ) {
            s = cDelta / ( cMax + cMin );
          } else {
            s = cDelta / ( 2.0 - ( cMax + cMin ) );
          }
          // hue
          if ( r == cMax ) {
            h = ( g - b ) / cDelta;
          } else if ( g == cMax ) {
            h = 2.0 + ( b - r ) / cDelta;
          } else {
            h = 4.0 + ( r - g ) / cDelta;
          }
          if ( h < 0.0) {
            h += 6.0;
          }
          h = h / 6.0;
        }
        return vec3( h, s, l );
      }
      // camera intrinsics from metadata file
      uniform float mindepth;
      uniform float maxdepth;
      uniform float imageWidth;
      uniform float imageHeight;
      uniform float focalX;
      uniform float focalY;
      uniform float principleX;
      uniform float principleY;
      uniform sampler2D map;
      varying float visibility;
      varying vec2 vUv;
      // projection formula
      vec3 xyz( float x , float y, float depth ) {
        float z = depth * ( maxdepth - mindepth ) + mindepth;
        return vec3(
       // The shader as it is written operates on a scale that seems about
       // 100x larger than units in AFRAME
       // Scale hack,
              0.01 * ((x - principleX)  * z / focalX) ,
              0.01 * ((y - principleY) * z / focalY),
              0.01 * (-z) );
      }
      void main() {
        vUv = vec2 (position.x/imageWidth, position.y/imageHeight);
        vUv.y = vUv.y * 0.5;// + 0.5;
        vec3 hsl = rgb2hsl( texture2D( map, vUv ).xyz );
        vec4 pos = vec4( xyz( position.x, position.y, hsl.x ), 5.0 );
        // Positioning the cloud
        // pos.x -= 07.0; Puts the base of the video in the right spot for rotated DepthKit video
        pos.z += 000.0;
        pos.y += 00.0;
        pos.x -= 07.0;
        visibility = hsl.z * 2.0;
        gl_PointSize = 2.0;
        gl_Position = projectionMatrix * modelViewMatrix * pos;
      }
    </script>

    <script id="fs" type="x-shader/x-fragment">
      uniform sampler2D map;
      uniform float opacity;
      varying float visibility;
      varying vec2 vUv;
      void main() {
        if ( visibility < 0.75 ) discard;
        vec4 color = texture2D( map, vUv + vec2(0.0, 0.5) );
        color.w = opacity;
        gl_FragColor = color;
      }
    </script>

<body>
  <a-scene>
  <a-entity rgbd-video="
         name:video;
         depthFocalLengthX: 365.2185974121094;
         depthFocalLengthY: 365.2185974121094;
         depthImageSizeX: 512;
         depthImageSizeY: 424;
         depthPrincipalPointX: 257.6430969238281;
         depthPrincipalPointY: 209.3957977294922;
         farClip: 1200;
         nearClip: 0;
         " position="0 .3 0" scale="1 1 1" rotation="0 0 -90"> </a-entity>
  </a-scene>
</body>
```
### Video Files
Videos should be exported per-pixel from DepthKit, then converted to .WebM and placed in a directory called 'rgbd_files' 
Follow this great tutorial for details on [exporting from DepthKit](https://vimeo.com/123520067)
