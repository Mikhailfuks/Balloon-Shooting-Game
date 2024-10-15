#include <SFML/Graphics.hpp>
#include <SFML/Audio.hpp>
#include <iostream>
#include <random>
#include <ctime>
#include <vector>

using namespace sf;
using namespace std;

// Constants
const int WINDOW_WIDTH = 800;
const int WINDOW_HEIGHT = 600;
const int BALLOON_SIZE = 50;
const int BALLOON_SPEED = 50;
const int NUM_BALLOONS = 10;

// Structures
struct Balloon {
    CircleShape shape;
    Vector2f velocity;
    bool isAlive;
};

// Function to generate a random position
Vector2f getRandomPosition() {
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<> distribX(BALLOON_SIZE, WINDOW_WIDTH - BALLOON_SIZE);
    uniform_int_distribution<> distribY(BALLOON_SIZE, WINDOW_HEIGHT - BALLOON_SIZE);
    return Vector2f(distribX(gen), distribY(gen));
}

// Function to generate a random color
Color getRandomColor() {
    random_device rd;
    mt19937 gen(rd());
    uniform_int_distribution<> distrib(0, 255);
    return Color(distrib(gen), distrib(gen), distrib(gen));
}

// Function to update the balloons
void updateBalloons(vector<Balloon>& balloons) {
    for (auto& balloon : balloons) {
        if (balloon.isAlive) {
            // Move the balloon
            balloon.shape.move(balloon.velocity);

            // Bounce off the walls
            if (balloon.shape.getPosition().x <= 0 || balloon.shape.getPosition().x + BALLOON_SIZE >= WINDOW_WIDTH) {
                balloon.velocity.x *= -1;
            }
            if (balloon.shape.getPosition().y <= 0 || balloon.shape.getPosition().y + BALLOON_SIZE >= WINDOW_HEIGHT) {
                balloon.velocity.y *= -1;
            }
        }
    }
}

// Function to check if a balloon is clicked
bool checkBalloonClick(const vector<Balloon>& balloons, const Vector2i& mousePos) {
    for (auto& balloon : balloons) {
        if (balloon.isAlive && balloon.shape.getGlobalBounds().contains(mousePos.x, mousePos.y)) {
            balloon.isAlive = false;
            return true;
        }
    }
    return false;
}

int main() {
    // Create the window
    RenderWindow window(VideoMode(WINDOW_WIDTH, WINDOW_HEIGHT), "Balloon Shooting Game");
    window.setFramerateLimit(60);

    // Load the sound effects
    SoundBuffer popSoundBuffer;
    if (!popSoundBuffer.loadFromFile("pop.wav")) {
        cerr << "Error loading pop sound" << endl;
        return 1;
    }
    Sound popSound;
    popSound.setBuffer(popSoundBuffer);

    // Create the balloons
    vector<Balloon> balloons;
    for (int i = 0; i < NUM_BALLOONS; ++i) {
        Balloon balloon;
        balloon.shape.setRadius(BALLOON_SIZE / 2);
        balloon.shape.setFillColor(getRandomColor());
        balloon.shape.setPosition(getRandomPosition());
        balloon.velocity = Vector2f(BALLOON_SPEED * ((rand() % 2) * 2 - 1), BALLOON_SPEED * ((rand() % 2) * 2 - 1));
        balloon.isAlive = true;
        balloons.push_back(balloon);
    }

    // Game loop
    while (window.isOpen()) {
        Event event;
        while (window.pollEvent(event)) {
            if (event.type == Event::Closed) {
                window.close();
            }
            if (event.type == Event::MouseButtonPressed) {
                if (event.mouseButton.button == Mouse::Left) {
                    if (checkBalloonClick(balloons, Mouse::getPosition(window))) {
                        popSound.play();
                    }
                }
            }
        }

        // Update the balloons
        updateBalloons(balloons);

        // Clear the window
        window.clear(Color::White);

        // Draw the balloons
        for (const auto& balloon : balloons) {
            if (balloon.isAlive) {
                window.draw(balloon.shape);
            }
        }

        // Display the window
        window.display();
    }

    return 0;
}
