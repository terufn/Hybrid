#include <GL/glew.h>
#include <GLFW/glfw3.h>
#include <vector>
#include <iostream>

struct Vertex {
    float x, y, z; // 頂点位置
    float r, g, b; // 色
};

class Mesh {
public:
    Mesh() : VBO(0), VAO(0) {}

    void create(const std::vector<Vertex>& vertices) {
        this->vertices = vertices;

        glGenVertexArrays(1, &VAO);
        glGenBuffers(1, &VBO);

        glBindVertexArray(VAO);

        glBindBuffer(GL_ARRAY_BUFFER, VBO);
        glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(Vertex), vertices.data(), GL_STATIC_DRAW);

        glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)0);
        glEnableVertexAttribArray(0);

        glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)(3 * sizeof(float)));
        glEnableVertexAttribArray(1);

        glBindVertexArray(0);
    }

    void update(const std::vector<Vertex>& newVertices) {
        this->vertices = newVertices;
        glBindBuffer(GL_ARRAY_BUFFER, VBO);
        glBufferSubData(GL_ARRAY_BUFFER, 0, vertices.size() * sizeof(Vertex), vertices.data());
    }

    void render() const {
        glBindVertexArray(VAO);
        glDrawArrays(GL_TRIANGLES, 0, vertices.size());
        glBindVertexArray(0);
    }

    ~Mesh() {
        glDeleteBuffers(1, &VBO);
        glDeleteVertexArrays(1, &VAO);
    }

private:
    GLuint VBO, VAO;
    std::vector<Vertex> vertices;
};

int main() {
    if (!glfwInit()) {
        std::cerr << "Failed to initialize GLFW\n";
        return -1;
    }

    GLFWwindow* window = glfwCreateWindow(800, 600, "Mesh Update Example", nullptr, nullptr);
    if (!window) {
        std::cerr << "Failed to create GLFW window\n";
        glfwTerminate();
        return -1;
    }
    glfwMakeContextCurrent(window);

    if (glewInit() != GLEW_OK) {
        std::cerr << "Failed to initialize GLEW\n";
        return -1;
    }

    std::vector<Vertex> initialVertices = {
        { -0.5f, -0.5f, 0.0f, 1.0f, 0.0f, 0.0f },
        {  0.5f, -0.5f, 0.0f, 0.0f, 1.0f, 0.0f },
        {  0.0f,  0.5f, 0.0f, 0.0f, 0.0f, 1.0f }
    };

    Mesh mesh;
    mesh.create(initialVertices);

    while (!glfwWindowShouldClose(window)) {
        glClear(GL_COLOR_BUFFER_BIT);


        std::vector<Vertex> updatedVertices = {
            { -0.5f, -0.5f, 0.0f, 1.0f, 0.5f, 0.0f },
            {  0.5f, -0.5f, 0.0f, 0.5f, 1.0f, 0.5f },
            {  0.0f,  0.5f, 0.0f, 0.5f, 0.5f, 1.0f }
        };
        mesh.update(updatedVertices);

        mesh.render();

        glfwSwapBuffers(window);
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
