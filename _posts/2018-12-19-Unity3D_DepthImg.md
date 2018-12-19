---
layout:     post
title:      "Unity3d DepthMap" 
subtitle:   ""
date:       2018-12-19 15:05:00
author:     "SiyuanWang"
#header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Unity3D 

---
# UWP Dynamic Tile
## 摄像机的代码 

        public class MainCameraShader : MonoBehaviour {

            public Material m_DepthMaterial;
            // Use this for initialization
            void Start () {
                
            }
            
            // Update is called once per frame
            void Update () {
                

            }
            void OnRenderImage(RenderTexture src, RenderTexture dest)
            {

                Graphics.Blit(src, dest,m_DepthMaterial);
            }



        }
## 给上面的代码加上shader 
        Shader "Custom/DepthMap" {
            SubShader{
                Tags{ "RenderType" = "Opaque" }

                Pass
                {

                    ZTest Always Cull Off ZWrite Off

                    CGPROGRAM

                    // Use shader model 3.0 target, to get nicer looking lighting
                    #pragma glsl
                    #pragma fragmentoption ARB_precision_hint_fastest
                    #pragma target 3.0
                    #pragma vertex vert
                    #pragma fragment frag
                    #include "unityCG.cginc"
                    sampler2D _CameraDepthTexture;

                    struct v2f 
                    {
                        float4 pos : SV_POSITION;
                        float4 scrPos:TEXCOORD0;
                    };
                    //Vertex Shader
                    v2f vert(appdata_base v) 
                    {
                        v2f o;
                        o.pos = UnityObjectToClipPos(v.vertex);
                        o.scrPos = ComputeScreenPos(o.pos);
                        return o;
                    }

                    //Fragment Shader
                    float4 frag(v2f i) :COLOR
                    {
                        float depthValue = 1 - Linear01Depth(tex2Dproj(_CameraDepthTexture, UNITY_PROJ_COORD(i.scrPos)).r);

                        int ar, ag, ab;
                        ar = depthValue * 0x1000000;
                        ag = ar % 65536;
                        ab = ar % 256;
                        ar /= 65536;
                        ag /= 256;

                        float dv = ab + ag * 256 + ar * 65536;
                        dv /= 0x1000000;
                        return float4(ar/256.0f, ag/256.0f, ab/256.0f, 1.0f);

                    }
                    
                    ENDCG
                }
            }
                FallBack "Diffuse"
        }