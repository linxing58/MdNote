# Cesium建筑自定义光源效果_cesium 添加光源_rpgpp55的博客-CSDN博客
![](https://img-blog.csdnimg.cn/1aada3c3f2fb46678940c8214eb55478.gif)

```javascript
  const customShader = new Cesium.CustomShader({
    lightingModel: Cesium.LightingModel.UNLIT,
    uniforms: {
      u_cameraDirectionWC: {
        type: Cesium.UniformType.VEC3,
        value: WE.viewer.scene.camera.positionWC,
      },
      u_lightColor1: {
        type: Cesium.UniformType.VEC4,
        value: lightPoint1.color,
      },
      u_lightPos1: {
        type: Cesium.UniformType.VEC3,
        value: lightPoint1.postion,
      },
      u_lightColor2: {
        type: Cesium.UniformType.VEC4,
        value: lightPoint2.color,
      },
      u_lightPos2: {
        type: Cesium.UniformType.VEC3,
        value: lightPoint2.postion,
      },
      u_lightColor3: {
        type: Cesium.UniformType.VEC4,
        value: lightPoint3.color,
      },
      u_lightPos3: {
        type: Cesium.UniformType.VEC3,
        value: lightPoint3.postion,
      },
    },
    fragmentShaderText: `
        vec4 makeLight(vec4 lightColorHdr,vec3 lightPos,
          vec3 positionWC,vec3 positionEC,vec3 normalEC,czm_pbrParameters pbrParameters)
        {
          vec3 color = vec3(0.0);
          float mx1 = 1.0;
          vec3 light1Dir = positionWC - lightPos;
          float distance1 = length(light1Dir);
          if(distance1 < 1000.0){
            vec4 l1 = czm_view * vec4(lightPos, 1.0);
            vec3 lightDirectionEC = l1.xyz - positionEC;
            mx1 = 1.0 - distance1 / 1000.0;
            color = czm_pbrLighting(
              positionEC,
              normalEC,
              lightDirectionEC,
              lightColorHdr.xyz,
              pbrParameters
            ).xyz;
          }
          mx1 = max(color.r,max(color.g,color.b)) * pow(mx1,1.0) * 10.0;
          return vec4(color,mx1);
        }
        void fragmentMain(FragmentInput fsInput, inout czm_modelMaterial material)
        {
          material.diffuse = vec3(1.0);
          vec3 positionWC = fsInput.attributes.positionWC;
          vec3 normalEC = fsInput.attributes.normalEC;
          vec3 positionEC = fsInput.attributes.positionEC;

          vec3 lightColorHdr = czm_lightColorHdr;
          vec3 lightDirectionEC = czm_lightDirectionEC;
          lightDirectionEC = (czm_view * vec4(u_cameraDirectionWC,1.0)).xyz - positionEC;

          czm_pbrParameters pbrParameters;
          pbrParameters.diffuseColor = material.diffuse;
          pbrParameters.f0 = vec3(0.5);
          pbrParameters.roughness = 1.0;

          vec3 ligth1Color0 = czm_pbrLighting(
            positionEC,
            normalEC,
            lightDirectionEC,
            lightColorHdr,
            pbrParameters
          );

          vec4 ligth1ColorR = makeLight(u_lightColor1,u_lightPos1,positionWC,positionEC,normalEC,pbrParameters);
          vec4 ligth1ColorG = makeLight(u_lightColor2,u_lightPos2,positionWC,positionEC,normalEC,pbrParameters);
          vec4 ligth1ColorB = makeLight(u_lightColor3,u_lightPos3,positionWC,positionEC,normalEC,pbrParameters);

          vec3 finalColor = mix(ligth1Color0.rgb, ligth1ColorR.rgb, ligth1ColorR.a);
          finalColor = mix(finalColor, ligth1ColorG.rgb, ligth1ColorG.a);
          finalColor = mix(finalColor, ligth1ColorB.rgb, ligth1ColorB.a);
          material.diffuse = finalColor;
        }
        `,
  });

```