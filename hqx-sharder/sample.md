# サンプルコードの解説

<pre>
このドキュメントは<a href="https://github.com/CrossVR/hqx-shader/blob/53540f5f0d985c385dc108b41ab89980f2b214f4/sample/main.cpp">sample/main.cpp/</a>を日本語で解説したものです。
</pre>

## ソースコード

```cpp
/* main.c
*
* Copyright (C) 2017 Jules Blok
*
* This software may be modified and distributed under the terms
* of the MIT license.  See the LICENSE file for details.
*/

#include <glad/glad.h>
#include <GLFW/glfw3.h>

#include "lodepng.h"
#include "linmath.h"

#include <string>
#include <iostream>
#include <fstream>
#include <cassert>

#ifdef _WIN32
#define _ "\\"
#else
#define _ "/"
#endif

static const struct
{
    float x, y, z, w;
    float u, v, s, t;
} vertices[] =
{
    { -1.f, -1.f, 0.f, 1.f, 0.f, 1.f, 0.f, 0.f },
    { -1.f,  1.f, 0.f, 1.f, 0.f, 0.f, 0.f, 0.f },
    {  1.f,  1.f, 0.f, 1.f, 1.f, 0.f, 0.f, 0.f },
    {  1.f, -1.f, 0.f, 1.f, 1.f, 1.f, 0.f, 0.f }
};

// 1xの頂点シェーダー
static const char* vertex_shader_text =
"attribute vec4 VertexCoord;\n"
"attribute vec4 TexCoord;\n"
"varying vec2 tex;\n"
"void main()\n"
"{\n"
"    gl_Position = VertexCoord;\n"
"    tex = TexCoord.xy;\n"
"}\n";

// 1xのフラグメントシェーダー
static const char* fragment_shader_text =
"uniform sampler2D Texture;\n"
"varying vec2 tex;\n"
"void main()\n"
"{\n"
"    gl_FragColor = texture2D(Texture, tex);\n"
"}\n";

// HQxのシェーダー
static const char* shader_files[] = {
    _"glsl" _"hq2x.glsl",
    _"glsl" _"hq3x.glsl",
    _"glsl" _"hq4x.glsl"
};

// LUTテクスチャ(シェーダ用の追加テクスチャ)
static const char* lut_files[] = {
    _"resources" _"hq2x.png",   // resource/hq2x.png 256x64
    _"resources" _"hq3x.png",   // resource/hq3x.png 256x144
    _"resources" _"hq4x.png"    // resource/hq4x.png 256x256
};

// インデックス指標(どの頂点を結んで線分を描くか)を保存する配列(IBO, EBO)
// 参照: https://qiita.com/y_UM4/items/8b87e82c66c185905553
static const uint8_t indices[] = { 0, 1, 2, 0, 2, 3 };

static uint32_t image_width, image_height, image_scale = 2;

static void error_callback(int error, const char* description)
{
    // ...
}

static void key_callback(GLFWwindow* window, int key, int scancode, int action, int mods)
{
    // ...
}

// ファイルをバイト列としてbufferに読み込む
static void read_file(const char* filename, std::vector<char>& buffer)
{
    std::ifstream file(filename, std::ios::ate);
    if (!file.is_open()) {
        std::cout << "Failed to open " << filename << std::endl;
        exit(EXIT_FAILURE);
    }

    std::streamsize size = file.tellg();
    file.seekg(0, std::ios::beg);

    buffer.resize(size);
    file.read(buffer.data(), size);
}

static GLuint load_texture(uint32_t* width, uint32_t* height, const char* filename)
{
    std::vector<uint8_t> image;
    uint32_t w, h, error;
    GLuint texture;

    error = lodepng::decode(image, w, h, filename);
    if (error) {
        error_callback(error, lodepng_error_text(error));
        exit(EXIT_FAILURE);
    }

    glGenTextures(1, &texture);
    glActiveTexture(GL_TEXTURE9); // loading stage
    glBindTexture(GL_TEXTURE_2D, texture);
    glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, w, h, 0, GL_RGBA, GL_UNSIGNED_BYTE, image.data());
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAX_LEVEL, 0);

    if (width) *width = w;
    if (height) *height = h;
    return texture;
}

static GLuint compile_shader(GLenum stage, const GLchar* source)
{
    GLchar* error_log;
    GLint compiled, length;
    GLuint shader;
    const GLchar* sources[2] = { "#version 130\n", source };

    // Both stages are present in the same file, use the pre-processor to separate them
    if (stage == GL_VERTEX_SHADER) sources[0] = "#version 130\n#define VERTEX\n";
    if (stage == GL_FRAGMENT_SHADER) sources[0] = "#version 130\n#define FRAGMENT\n";

    shader = glCreateShader(stage);
    glShaderSource(shader, 2, sources, NULL);
    glCompileShader(shader);

    glGetShaderiv(shader, GL_COMPILE_STATUS, &compiled);
    if (compiled == GL_FALSE) {
        glGetShaderiv(shader, GL_INFO_LOG_LENGTH, &length);
        error_log = new char[length];
        glGetShaderInfoLog(shader, length, &length, error_log);
        error_callback(GL_INVALID_OPERATION, error_log);
        delete error_log;
    }

    return shader;
}

// シェーダープログラムを作成
static GLuint link_program(GLuint vertex_shader, GLuint fragment_shader)
{
    GLchar* error_log;
    GLint compiled, length;
    GLuint program;

    program = glCreateProgram();
    glAttachShader(program, vertex_shader);
    glAttachShader(program, fragment_shader);
    glLinkProgram(program);

    glGetProgramiv(program, GL_LINK_STATUS, (int *)&compiled);
    if (compiled == GL_FALSE) {
        glGetShaderiv(program, GL_INFO_LOG_LENGTH, &length);
        error_log = new char[length];
        glGetProgramInfoLog(program, length, &length, error_log);
        error_callback(GL_INVALID_OPERATION, error_log);
        delete error_log;
    }

    // We don't need the shaders anymore
    glDeleteShader(vertex_shader);
    glDeleteShader(fragment_shader);

    return program;
}

int main(int argc, const char* argv[])
{
    if (argc < 2) {
        std::cout << "Usage: " << argv[0] << " <hqx-shader folder> [image file]" << std::endl;
        exit(EXIT_FAILURE);
    }

    // Set up some basic paths based on the input arguments
    std::string base_path = argv[1];
    std::string image_path(base_path);
    if (argc > 2)
        image_path = argv[2];
    else
        image_path.append(_"sample" _"pixelart0.png");

    // Initialise GLFW and create the window
    glfwSetErrorCallback(error_callback);
    if (!glfwInit()) exit(EXIT_FAILURE);

    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 2);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 0);

    GLFWwindow* window = glfwCreateWindow(640, 480, "HQx Sample", NULL, NULL);
    if (!window) {
        glfwTerminate();
        exit(EXIT_FAILURE);
    }

    glfwSetKeyCallback(window, key_callback);

    glfwMakeContextCurrent(window);
    gladLoadGLLoader((GLADloadproc) glfwGetProcAddress);
    glfwSwapInterval(1);

    // アップスケールする画像をテクスチャ(変数`texture`)として読み込む
    GLuint texture = load_texture(&image_width, &image_height, image_path.c_str()); // image_path: アップスケール対象の画像 e.g. sample/pixelart0.png
    glActiveTexture(GL_TEXTURE0);
    glBindTexture(GL_TEXTURE_2D, texture);

    // 頂点バッファ(VBO)にフルスクリーンの四角形を読み込む
    GLuint vertex_buffer;
    glGenBuffers(1, &vertex_buffer);
    glBindBuffer(GL_ARRAY_BUFFER, vertex_buffer);
    glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);  // 頂点バッファオブジェクトのメモリを確保し、そこにデータ (頂点属性) を転送

    // すべてのアップスケール(HQx)シェーダーを含むベクタを初期化する
    // ベクタのインデックスはスケール倍率を表す
    std::vector<GLuint> programs, lut_textures;
    programs.push_back(NULL);       // programs:     シェーダープログラムを格納するベクタ
    lut_textures.push_back(NULL);   // lut_textures: 

    // 何もしないシェーダー(1xシェーダー)をロードする
    GLuint vertex_shader = compile_shader(GL_VERTEX_SHADER, vertex_shader_text);
    GLuint fragment_shader = compile_shader(GL_FRAGMENT_SHADER, fragment_shader_text);
    programs.push_back(link_program(vertex_shader, fragment_shader));
    lut_textures.push_back(NULL); // no lookup table set

    // 1xシェーダーのUniform変数を設定
    glUniform1i(glGetUniformLocation(programs[1], "Texture"), 0);
    GLint vpos_location = glGetAttribLocation(programs[1], "VertexCoord");
    GLint vtex_location = glGetAttribLocation(programs[1], "TexCoord");

    // attribute変数を設定。
    // すべてのシェーダーは互換性のあるシグネチャを持っているので、この作業は一度しか行われない
    glEnableVertexAttribArray(vpos_location);
    glVertexAttribPointer(vpos_location, 4, GL_FLOAT, GL_FALSE, sizeof(vertices[0]), (void*)0);
    glEnableVertexAttribArray(vtex_location);
    glVertexAttribPointer(vtex_location, 4, GL_FLOAT, GL_FALSE, sizeof(vertices[0]), (void*)(sizeof(float) * 4));

    // アップスケールシェーダー(HQx)をロードする
    mat4x4 mvp;
    mat4x4_identity(mvp);
    for (int i = 0; i < 3; i++) {
        std::vector<char> shader;               // HQxシェーダーのソースコード    
        std::string shader_path(base_path);
        shader_path.append(shader_files[i]);    // e.g. shader_path = glsl/hq2x.glsl

        // HQxシェーダープログラム(program)の作成
        read_file(shader_path.c_str(), shader);
        vertex_shader = compile_shader(GL_VERTEX_SHADER, shader.data());
        fragment_shader = compile_shader(GL_FRAGMENT_SHADER, shader.data());
        GLuint program = link_program(vertex_shader, fragment_shader);

        // HQxシェーダーのUniform変数ポインタを取得
        GLint mvp_location = glGetUniformLocation(program, "MVPMatrix");
        GLint samp_location = glGetUniformLocation(program, "Texture");
        GLint lut_location = glGetUniformLocation(program, "LUT");
        GLint tsize_location = glGetUniformLocation(program, "TextureSize");

        // HQxシェーダーのUniform変数を設定
        glUseProgram(program);
        glUniformMatrix4fv(mvp_location, 1, GL_FALSE, (const GLfloat*)mvp);
        glUniform1i(samp_location, 0);
        glUniform1i(lut_location, 1);
        glUniform2f(tsize_location, (float)image_width, (float)image_height);

        // LUTテクスチャのロード
        std::string lut_path(base_path);
        lut_path.append(lut_files[i]);          // e.g. lut_path = resource/hq2x.png
        GLuint lut = load_texture(nullptr, nullptr, lut_path.c_str());

        // ベクタにpush
        programs.push_back(program);
        lut_textures.push_back(lut);
    }

    // ウィンドウのサイズをデフォルトの縮尺(画像サイズの2倍)に変更し、レンダリングループに入る
    glfwSetWindowSize(window, image_width * image_scale, image_height * image_scale);
    while (!glfwWindowShouldClose(window)) {
        // 指定されたウィンドウに割り当てられているフレームバッファのサイズを調べる
        int width, height;
        glfwGetFramebufferSize(window, &width, &height);

        glViewport(0, 0, width, height);    // ビューポート: コンピューターグラフィックの中で現在表示されている領域
        glClear(GL_COLOR_BUFFER_BIT);

        // アップスケールテクスチャの適用
        glUseProgram(programs[image_scale]);
        glActiveTexture(GL_TEXTURE1);           // 事前にGL_TEXTURE0にスケール対象の画像を読み込んでいる
        glBindTexture(GL_TEXTURE_2D, lut_textures[image_scale]);

        // 四角形(ゲームスクリーン)の描画
        // 三角形(GL_TRIANGLES)2つの描画で四角形を表現するので必要な頂点数(第2引数)は6
        glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_BYTE, indices);

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwDestroyWindow(window);

    glfwTerminate();
    exit(EXIT_SUCCESS);
}
```