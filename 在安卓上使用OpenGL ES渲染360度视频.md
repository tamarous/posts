# 在安卓上使用OpenGL ES渲染360度视频

随着VR及AR概念的火热，在安卓手机上显示和观看360度视频成为了一个常见的需求。本篇文章介绍了我在开发一款安卓VR播放器时遇到的一些问题及相应的解决方案。

## 项目描述
我做的是一款VR直播播放器，这个项目的流程是这样的：首先通过360度摄像头采集整个场景的信息，然后通过转码平台实时地进行编解码并生成MPD文件，播放器通过读取服务器上的MPD文件来进行播放。
在做这个播放器的时候，我负责的模块主要是与OpenGL ES相关的，遇到了不少坑，比如显示不出来图像、绿屏等现象，好在最后都解决了。

## 360度视频显示原理
我们的播放器可以播放画幅比例为2：1的，采用Equirectangular Projection的视频。这种视频由于水平方向上是360度，垂直方向上是180度，因此渲染时可以当做纹理贴到一个球体上。因此整个显示过程可以分为几个步骤：初始化OpenGL、设置球体坐标及纹理坐标、设置纹理数据、绘制等过程。当然这只是一个粗略的过程，在每一个步骤中还有很多准备工作，由于本文不是介绍OpenGL的使用的，因此略过不谈。

### 设置球体坐标及纹理坐标
- 为存储球体坐标和纹理坐标的矩阵分配空间
由于在OpenGL中没有办法绘制出标准的球形，因此常用的做法是将球形细分成三角形面片，当面片的数量足够多时，看起来就比较接近于球形了。但是如果这个数字太大，就会导致绘制球形的时间过长，引起帧率下降、占用内存过大等问题，因此需要做一个合理的折中。我们将这个数字设置为了64。

```
GLHelper::GLHelper() {
    this->numberOfPatches = 64;
    this->vertexCount = this->numberOfPatches * this->numberOfPatches/2 * 6;
    this->vertexCoordinates = (float *)malloc(this->vertexCount * 3 *sizeof(float));
    this->uvCoordinates = (float *)malloc(this->vertexCount * 2 * sizeof(float));
}
```

- 设置球体及纹理坐标

```
bool GLHelper::setupVertices() {
    int radius = 10; // 球的半径
    int pieces = this->numberOfPatches; 
    int half_pieces = this->numberOfPatches/2;
    double step_z = M_PI/(half_pieces); 
    double step_xy = step_z;

    double angle_z;
    double angle_xy;

    float z[4] = {0.0f};
    float x[4] = {0.0f};
    float y[4] = {0.0f};
    float u[4] = {0.0f};
    float v[4] = {0.0f};

    int m = 0, n = 0;
    for(int i = 0; i < half_pieces; i++) {

        angle_z = i * step_z;
        for(int j = 0; j < pieces;j ++ ) {
            angle_xy = j * step_xy;

            z[0] = (float)(radius * sin(angle_z)*cos(angle_xy));
            x[0]= (float)(radius*sin(angle_z)*sin(angle_xy));
            y[0]= (float)(radius*cos(angle_z));
            u[0]= (float)j / pieces;
            v[0]= (float) i/half_pieces;

            z[1] = (float)(radius*sin(angle_z+step_z)*cos(angle_xy));
            x[1] = (float)(radius*sin(angle_z + step_z)*sin(angle_xy));
            y[1] = (float)(radius*cos(angle_z+step_z));
            u[1] = (float)j/pieces;
            v[1] = (float)(i+1)/half_pieces;

            z[2] = (float)(radius*sin(angle_z+step_z)*cos(angle_xy+step_xy));
            x[2] = (float)(radius *sin(angle_z+step_z)*sin(angle_xy+step_xy));
            y[2] = (float)(radius*cos(angle_z+step_z));
            u[2] = (float)(j+1)/pieces;
            v[2] = (float)(i+1)/half_pieces;

            z[3] = (float)(radius*sin(angle_z)*cos(angle_xy+step_xy));
            x[3] = (float)(radius*sin(angle_z)*sin(angle_xy+step_xy));
            y[3] = (float)(radius*cos(angle_z));
            u[3] = (float)(j+1)/pieces;
            v[3] = (float)i/half_pieces;

            this->vertexCoordinates[m++] = x[0];
            this->vertexCoordinates[m++] = y[0];
            this->vertexCoordinates[m++] = z[0];
            this->uvCoordinates[n++] = u[0];
            this->uvCoordinates[n++] = v[0];

            this->vertexCoordinates[m++] = x[1];
            this->vertexCoordinates[m++] = y[1];
            this->vertexCoordinates[m++] = z[1];
            this->uvCoordinates[n++] = u[1];
            this->uvCoordinates[n++] = v[1];

            this->vertexCoordinates[m++] = x[2];
            this->vertexCoordinates[m++] = y[2];
            this->vertexCoordinates[m++] = z[2];
            this->uvCoordinates[n++] = u[2];
            this->uvCoordinates[n++] = v[2];

            this->vertexCoordinates[m++] = x[2];
            this->vertexCoordinates[m++] = y[2];
            this->vertexCoordinates[m++] = z[2];
            this->uvCoordinates[n++] = u[2];
            this->uvCoordinates[n++] = v[2];

            this->vertexCoordinates[m++] = x[3];
            this->vertexCoordinates[m++] = y[3];
            this->vertexCoordinates[m++] = z[3];
            this->uvCoordinates[n++] = u[3];
            this->uvCoordinates[n++] = v[3];

            this->vertexCoordinates[m++] = x[0];
            this->vertexCoordinates[m++] = y[0];
            this->vertexCoordinates[m++] = z[0];
            this->uvCoordinates[n++] = u[0];
            this->uvCoordinates[n++] = v[0];
        }
    }

    // 将索引号和着色器中的变量联系起来
    gPositionAttribPointer = glGetAttribLocation(gProgram, "in_pos");
    gSamplerAttribPointer = glGetAttribLocation(gProgram, "in_tc");
    gMatrixUniformPointer = glGetUniformLocation(gProgram,"matrix");
    glCheckError();
    return true;
}
```

### 着色器设置

安卓MediaCodec框架解码视频帧得到的格式一般是YUV的，而OpenGL中不能直接将YUV格式的数据当做纹理，
所以我们必须把YUV格式的数据转换为OpenGL支持的RGB24格式。但是由于YUV格式数据量很大，以我们所用的4K视频为例，一帧视频的数据量有：`3840*1920*1.5 = 11059200 Byte= 10.55MB`，因此如果在CPU上做转换，那么会导致我们的可用渲染时长被大大压缩，甚至有可能导致帧率下降、画面卡顿等严重影响用户体验的问题。我们最后的解决办法是在OpenGL着色器中进行YUV到RGB的转换，这样颜色转换就会在手机的GPU中进行，不占用CPU运算时间。

顶点着色器所做的事情很简单：一是将matrix表示的MVP矩阵和球体上各点坐标相乘得到最后显示在屏幕上的点坐标，二是把纹理坐标传递到片段着色器中。

```
static const char VERTEX_SHADER[] =
        "uniform mat4 matrix;\n"
        "varying vec2 interp_tc;\n"
        "attribute vec4 in_pos;\n"
        "attribute vec2 in_tc;\n"
        "void main() {\n"
        "  gl_Position = matrix * in_pos;\n"
        "  interp_tc = in_tc;\n"
        "}\n";
```

片段着色器：使用纹理坐标分别对y,u,v三个通道的数据进行采样，最后计算得到rgb颜色。

```
static const char FRAGMENT_SHADER[] =
        "precision mediump float;\n"
        "varying vec2 interp_tc;\n"
        "uniform sampler2D y_tex;\n"
        "uniform sampler2D u_tex;\n"
        "uniform sampler2D v_tex;\n"
        "void main() {\n"
        "  float y = 1.164 * (texture2D(y_tex, interp_tc).r - 0.0625);\n"
        "  float u = texture2D(u_tex, interp_tc).r - 0.5;\n"
        "  float v = texture2D(v_tex, interp_tc).r - 0.5;\n"
        "  gl_FragColor = vec4(y + 1.596 * v, "
        "                      y - 0.391 * u - 0.813 * v, "
        "                      y + 2.018 * u, "
        "                      1.0);\n"
        "}\n";
```  

### 设置纹理及数据
我们创建三个纹理坐标号，分别对应Y、U、V三个通道的数据。

```
const char* TEXTURE_UNIFORMS[] = {"y_tex", "u_tex", "v_tex"};
GLuint yuvTextures[3];

bool GLHelper::setupTextures() {
    glUseProgram(gProgram);
    glGenTextures(3, yuvTextures);
    for (int i = 0; i < 3; i++)  {
        glUniform1i(glGetUniformLocation(gProgram, TEXTURE_UNIFORMS[i]), i);
        glActiveTexture(GL_TEXTURE0 + i);
        glBindTexture(GL_TEXTURE_2D, yuvTextures[i]);
        glTexParameterf(GL_TEXTURE_2D,
                        GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameterf(GL_TEXTURE_2D,
                        GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        glTexParameterf(GL_TEXTURE_2D,
                        GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameterf(GL_TEXTURE_2D,
                        GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    }
    glUseProgram(0);
    glCheckError();
    return true;
}
```

### 绘制

```
void GLHelper::drawFrame(int frameWidth, int frameHeight, unsigned char *YData,
                         unsigned char *UData, unsigned char *VData) {
    glViewport(0, 0, screenWidth, screenHeight);
    glUseProgram(gProgram);
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);
    glCheckError();
    glVertexAttribPointer(gPositionAttribPointer,3,GL_FLOAT,GL_FALSE,0,this->vertexCoordinates);
    glEnableVertexAttribArray(gPositionAttribPointer);
    glCheckError();
    glVertexAttribPointer(gSamplerAttribPointer,2,GL_FLOAT,GL_FALSE,0,this->uvCoordinates);
    glEnableVertexAttribArray(gSamplerAttribPointer);

    glUniformMatrix4fv(gMatrixUniformPointer,1,GL_FALSE,mvpMatrix);
    
    unsigned char *yuvPlanes[3];
    yuvPlanes[0] = YData;
    yuvPlanes[1] = UData;
    yuvPlanes[2] = VData;
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
    
    for (int i = 0; i < 3; i++) {
        int h = (i == 0) ? frameHeight : frameHeight/ 2;
        int w = (i == 0) ? frameWidth : frameWidth/2;
        glActiveTexture(GL_TEXTURE0 + i);
        glBindTexture(GL_TEXTURE_2D, yuvTextures[i]);
        glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, w,
                     h, 0, GL_LUMINANCE, GL_UNSIGNED_BYTE, yuvPlanes[i]);
        
    }
    
    glDrawArrays(GL_TRIANGLES,0,this->vertexCount);
    eglSwapBuffers(dpy, surface);
}
```

## 双目显示原理
上面的代码是以普通播放器的平面方式显示视频的。如果我们需要以VR应用中常见的双目形式显示视频，那么需要借助于`Render To Texture`技术。

关于`Render To Texture`技术的原理，可以参考以下链接：[Framebuffer Object](https://learnopengl-cn.github.io/04%20Advanced%20OpenGL/05%20Framebuffers/)。

![VR Video](http://7xnyik.com1.z0.glb.clouddn.com/vr.jpg)

双目显示的过程如下：

### 创建帧缓冲区
我们平常使用的渲染终点都是屏幕提供的默认缓冲区，由于绘制每一帧时要在屏幕的左右两边显示具有一定视角差异的两幅图像，因此默认缓冲区不能满足我们的要求，我们需要创建两个自定义的帧缓冲区，分别对应左右两只眼睛的视角，然后将左右两边的图像分别渲染到这两个缓冲区中。

```

struct FramebufferDesc {
    // Depth Buffer Index used to represent depth component.
    GLuint m_nDepthBufferID;
    // Render Buffer Index used to represent color component.
    GLuint m_nRenderTextureID;
    // FrameBuffer object Index used to represent framebuffer object.
    GLuint m_nRenderFramebufferID;
};

bool GLHelper::setupFrameBuffer(int screenWidth, int screenHeight, FramebufferDesc &framebufferDesc) {
   
    // Generate Framebuffer Object ID
    glGenFramebuffers(1, &framebufferDesc.m_nRenderFramebufferID);
    glGenTextures(1, &framebufferDesc.m_nRenderTextureID);
    glGenRenderbuffers(1, &framebufferDesc.m_nDepthBufferID);

    // Generate Texture and setup Texture Parameters
    glBindTexture(GL_TEXTURE_2D,framebufferDesc.m_nRenderTextureID);

    // Framebuffer 里的Texture的宽高通常为屏幕的宽高
    glTexImage2D(GL_TEXTURE_2D,0,GL_RGB,screenWidth,screenHeight,0,GL_RGB,GL_UNSIGNED_BYTE,NULL);

    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MIN_FILTER,GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_MAG_FILTER,GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D,GL_TEXTURE_WRAP_T,GL_CLAMP_TO_EDGE);

    glBindRenderbuffer(GL_RENDERBUFFER,framebufferDesc.m_nDepthBufferID);
                                                                                                
    glRenderbufferStorage(GL_RENDERBUFFRER, GL_DEPTH_COMPONENT16, screenWidth, screenHeight);

    glBindFramebuffer(GL_FRAMEBUFFER, framebufferDesc.m_nRenderFramebufferID);
    
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, framebufferDesc.m_nRenderTextureID, 0);
    
    glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_RENDERBUFFER, framebufferDesc.m_nDepthBufferID);

    GLenum status = glCheckFramebufferStatus(GL_FRAMEBUFFER);
    if (status != GL_FRAMEBUFFER_COMPLETE) {
        LOGE("Error in creating framebuffer object");
        return false;
    }

    glBindFramebuffer(GL_FRAMEBUFFER, 0);
    return true;
}
```
### 绘制
在绘制每一帧时，分为两个子过程：一是在左右帧缓冲区中将具有视差的两幅图像当做纹理渲染到球体上并形成一幅新的纹理，二是将在默认缓冲区中将上一步中生成的纹理渲染到屏幕的左右半边对应的矩形上。这两个过程分别对应着`renderToTexture()`和`renderToScreen()`。

```
void GLHelper::renderToTexture(int yuvFrameWidth, int yuvFrameHeight, unsigned char *YData,
                               unsigned char *UData, unsigned char *VData, EYE eye) {
                               
    // 1. 绑定到对应的缓冲区上
    if (eye == LEFT_EYE) {
        glBindFramebuffer(GL_FRAMEBUFFER, leftEyeFrameBufferDesc.m_nRenderFramebufferID);
    } else if (eye == RIGHT_EYE) {
        glBindFramebuffer(GL_FRAMEBUFFER, rightEyeFrameBufferDesc.m_nRenderFramebufferID);
    }

    // 2. 使用绘制球体的着色器
    glUseProgram(sphereProgram);

    // 3. 计算MVP矩阵
    glm::mat4 mvpMat = myCamera->GetMVP(eye);
    float mat[16] = {0.0f};
    const float *pSrc = (const float *)glm::value_ptr(mvpMat);
    for(int i = 0; i < 16;i++) {
        mat[i] = pSrc[i];
    }

    glUniformMatrix4fv(gSphereMatrixUniformPointer,1,GL_FALSE,mat);
    
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT);

    glVertexAttribPointer(gSpherePositionAttribPointer,3,GL_FLOAT,GL_FALSE,0,this->vertexCoordinates);
    glEnableVertexAttribArray(gSpherePositionAttribPointer);

    glVertexAttribPointer(gSphereSamplerAttribPointer,2,GL_FLOAT,GL_FALSE,0,this->uvCoordinates);
    glEnableVertexAttribArray(gSphereSamplerAttribPointer);
    
    unsigned char *yuvPlanes[3];
    yuvPlanes[0] = YData;
    yuvPlanes[1] = UData;
    yuvPlanes[2] = VData;
    glViewport(0,0,w,h);
    glPixelStorei(GL_UNPACK_ALIGNMENT,1);
    for (int i = 0; i < 3; i++) {
        int w = (i == 0 ? yuvFrameWidth  : yuvFrameWidth / 2);
        int h = (i == 0 ? yuvFrameHeight : yuvFrameHeight / 2);
        glActiveTexture(GL_TEXTURE0 + i);
        glBindTexture(GL_TEXTURE_2D, yuvTextures[i]);
        glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, w, h, 0, GL_LUMINANCE, GL_UNSIGNED_BYTE, yuvPlanes[i]);
    }

    glDrawArrays(GL_TRIANGLES,0,this->vertexCount);
    glBindTexture(GL_TEXTURE_2D,0);
    
    // 4. 切换到默认缓冲区上
    glBindFramebuffer(GL_FRAMEBUFFER,0);
    glUseProgram(0);
}


void GLHelper::renderToScreen() {
    glClearColor(0.0f, 0.0f, 0.0f, 1.0f);
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
    glViewport(0,0,w,h);

    // 切换到左半边矩形对应的着色器
    glUseProgram(leftScreenProgram);
    gLeftScreenPositionAttribPointer = glGetAttribLocation(leftScreenProgram, "in_pos");
    glEnableVertexAttribArray(gLeftScreenPositionAttribPointer);
    glVertexAttribPointer(gLeftScreenPositionAttribPointer,2,GL_FLOAT,GL_FALSE,0,LEFTSCREEN_VERTICES);
    gLeftScreenSamplerAttribPointer = glGetAttribLocation(leftScreenProgram, "in_tc");
    glEnableVertexAttribArray(gLeftScreenSamplerAttribPointer);
    glVertexAttribPointer(gLeftScreenSamplerAttribPointer,2,GL_FLOAT,GL_FALSE,0,SAMPLER_VERTICES);
    glUniform1i(glGetUniformLocation(leftScreenProgram,"texture"),3);
    glActiveTexture(GL_TEXTURE3);
    glBindTexture(GL_TEXTURE_2D, leftEyeFrameBufferDesc.m_nRenderTextureID);
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    glBindTexture(GL_TEXTURE_2D, 0);
    glUseProgram(0);
   
    // 切换到右半边矩形对应的着色器
    glUseProgram(rightScreenProgram);
    gRightScreenPositionAttribPointer = glGetAttribLocation(rightScreenProgram, "in_pos");
    glEnableVertexAttribArray(gRightScreenPositionAttribPointer);
    glVertexAttribPointer(gRightScreenPositionAttribPointer, 2, GL_FLOAT, GL_FALSE, 0, RIGHTSCREEN_VERTICES);
    gRightScreenSamplerAttribPointer = glGetAttribLocation(rightScreenProgram, "in_tc");
    glEnableVertexAttribArray(gRightScreenSamplerAttribPointer);
    glVertexAttribPointer(gRightScreenSamplerAttribPointer,2,GL_FLOAT,GL_FALSE,0,SAMPLER_VERTICES);
    glUniform1i(glGetUniformLocation(rightScreenProgram,"texture"),4);
    glActiveTexture(GL_TEXTURE4);
    glBindTexture(GL_TEXTURE_2D, rightEyeFrameBufferDesc.m_nRenderTextureID);
    glDrawArrays(GL_TRIANGLE_STRIP, 0, 4);
    glBindTexture(GL_TEXTURE_2D, 0);
    glUseProgram(0);
    
}
```
### 调试
某些时候，不小心传入了错误的参数，或者是忘记进行某些状态设置，会导致绿屏、黑屏等不正确的显示效果，而这些错误原因往往难以察觉，给我们的项目顺利进展带来一些阻碍。这时候，我们可以通过`glGetError()`来获知上一句OpenGL函数调用有没有引起错误，从而一点一点定位问题所在。
我对这个函数进行了一些封装，让它的使用更为方便：

```
#define glCheckError() glCheckError_(__LINE__)

void glCheckError_(int line)
{
    GLenum errorCode;
    char error[100];
    memset(error,0,sizeof(error));
    while ((errorCode = glGetError()) != GL_NO_ERROR)
    {

        switch (errorCode)
        {
            case GL_INVALID_ENUM:                  sprintf(error,"GL_INVALID_ENUM"); break;
            case GL_INVALID_VALUE:                 sprintf(error,"GL_INVALID_VALUE"); break;
            case GL_INVALID_OPERATION:             sprintf(error,"GL_INVALID_OPERATION"); break;
            case GL_OUT_OF_MEMORY:                 sprintf(error,"GL_OUT_OF_MEMORY"); break;
            case GL_INVALID_FRAMEBUFFER_OPERATION: sprintf(error,"GL_INVALID_FRAMEBUFFER_OPERATION"); break;
        }
        LOGE("Line is %d, glError: %s", line, error);
    }
}
```
在我们怀疑可能出错的函数调用后加上这么一句，如果有错就会在控制台打印出错误原因，便于我们更快地找出问题。

```
    glTexImage2D(GL_TEXTURE_2D, 0, GL_LUMINANCE, w, h, 0, GL_LUMINANCE,     GL_UNSIGNED_BYTE, yuvPlanes[i]);
    glCheckError();
```


### 小结
完整的代码可以查看[GLHelper.h](https://gist.github.com/tamarous/16ff3577979596d93da437778b9ff898)和[GLHelper.cpp](https://gist.github.com/tamarous/7250ba7c1e7f6e634228b313025f6c3d)。





 









