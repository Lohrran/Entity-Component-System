# Entity Component System
> A tiny ECS C++ framework


## Why?
After I read the book **[Game Programming Patterns by Robert Nystrom](https://gameprogrammingpatterns.com)**, I could not stop think in ways to apply the different designs on the book, however for all of them, ECS called more my atention. So here it is my attempt of implement it and learn a bit more about C++


## Introduction
It's a framework simple to understand and easy to use, it was based on different articles (references below) from cross internet and the book Game Programming Patterns.
For the communication between the *systems* was implement **[Publisher / Subscribe Pattern](https://en.wikipedia.org/wiki/Publish–subscribe_pattern)**, based on in this [forum post](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/effective-event-handling-in-c-r2459).

> So the framework have some issues handling memory allocation although I maybe had managed it well for my first attempt using C++ 😎/😭.


## What it Looks Like?
The below implementation was made using the SDL2 library, you can also check it in the **[Tiny-Engine-2D](https://github.com/Lohrran/Tiny-Engine-2D/blob/master/README.md)**:

```c++

#define FRAMES_PER_SECOND 60
#define SCREEN_WIDTH 600
#define SCREEN_HEIGHT 400

//Standard Library
#include <iostream>

//SDL2 Library
#include <SDL.h>

//ECS
#include "Scene.h"
#include "Resources.h"

//Components
#include "ScreenComponent.h"
#include "SpriteComponent.h"
#include "PositionComponent.h"
#include "TagComponent.h"

//Systems
#include "ScreenSystem.h"
#include "SpriteSystem.h"
#include "MovementSystem.h"

int main(int argc, char *argv[])
{
	//Temporary helper to Loop the Game
	bool quit = false;
	SDL_Event event;

	/* --- SCENE ---*/
	
	Scene scene;
	Resources resources{ &scene };

	//Game Screen
	GameObject* screen = scene.createGameObject();
	screen.addComponent<ScreenComponent>("Game", SCREEN_WIDTH, SCREEN_HEIGHT);

	/* --- GAMEOBJECTS ---*/
	
	GameObject* player = scene.createGameObject();
	player->addComponent<SpriteComponent>("C:\\Users\\user\\Desktop\\player_34x34.bmp", 34, 34);
	player->addComponent<PositionComponent>(50, 50);
	player->addComponent<TagComponent>("Player");

	GameObject* enemy = scene.createGameObject();
	enemy->addComponent<SpriteComponent>("C:\\Users\\user\\Desktop\\enemy_34x34.bmp", 34, 34);
	enemy->addComponent<PositionComponent>(300, 100);
	enemy->addComponent<TagComponent>("Enemy");


	/* --- SYSTEMS --- */

	resources.add<ScreenSystem>();
	resources.add<SpriteSystem>();
	resources.add<MovementSystem>();


	/* --- RUNNING THE GAME --- */

	//Start
	resources.init<ScreenSystem>();
	resources.init<SpriteSystem>();
	resources.init<MovementSystem>();

	//Update
	while (!quit)
	{
		while (SDL_PollEvent(&event) != 0)
		{
			//User requests quit
			if (event.type == SDL_QUIT)
			{
				quit = true;
			}
		}
				
		resources.update<SpriteSystem>();
		resources.update<MovementSystem>();
		resources.update<ScreenSystem>();

		SDL_Delay(1000 / FRAMES_PER_SECOND);
	}

	//Clean
	resources.free<SpriteSystem>();
	resources.free<MovementSystem>();
	resources.free<ScreenSystem>();

	std::cout << "\n\n\n";
	system("pause");
	return 0;
}

```


## How to

Import the files for your project.


#### Create Scene and Resource

```c++

Scene scene;
Resources resources { &scene };

```

#### Create Game Object

```c++

GameObject* player = scene.createGameObject();

```


#### Create Component

```c++
#include "Component.h"

struct PositionComponent : public Component
{
	PositionComponent(int x = 0, int y = 0) : x{ x }, y{ y } { }

	int x, y;
};
```


#### Create Event

```c++
//Entity Component System
#include "Event.h"
#include "GameObject.h"

struct StateEvent : public Event
{
	StateEvent(GameObject* obj, std::string state) : obj{ obj }, state{ state } { }

	GameObject* obj;
	std::string state;
};
```

#### Create System

##### .h File:

```c++
//Entity Component System
#include "Requeriment.h"

//Components
#include "PositionComponent.h"
#include "VelocityComponent.h"
#include "DirectionComponent.h"

//Event
#include "StateEvent.h"

class MovementSystem : public Requeriment <PositionComponent, VelocityComponent, DirectionComponent>
{
	public:
		void init(GameObject* obj) override;
		void update(GameObject* obj)override;
		void free(GameObject* obj) override;
};
```

##### .cpp File:
```C++
#include "MovementSystem.h"

MovementSystem::MovementSystem() { }

void MovementSystem::init(GameObject* obj)
{
	obj->getComponent<PositionComponent>()->x = obj->getComponent<PositionComponent>()->x;
	obj->getComponent<PositionComponent>()->y = obj->getComponent<PositionComponent>()->y;
}

void MovementSystem::update(GameObject* obj)
{
	switch (obj->getComponent<DirectionComponent>()->direction)
	{
		case Direction::UP:
			obj->getComponent<PositionComponent>()->y -= obj->getComponent<VelocityComponent>()->y;
			eventChannel->publish(new StateEvent(obj, "WALK UP"));
			break;
		
		case Direction::DOWN:
			obj->getComponent<PositionComponent>()->y += obj->getComponent<VelocityComponent>()->y;
			eventChannel->publish(new StateEvent(obj, "WALK DOWN"));
			break;

		case Direction::RIGHT:
			obj->getComponent<PositionComponent>()->x += obj->getComponent<VelocityComponent>()->x;
			eventChannel->publish(new StateEvent(obj, "WALK RIGHT"));
			break;
		
		case Direction::LEFT:
			obj->getComponent<PositionComponent>()->x -= obj->getComponent<VelocityComponent>()->x;
			eventChannel->publish(new StateEvent(obj, "WALK LEFT"));
			break;

		default:
			obj->getComponent<PositionComponent>()->x = obj->getComponent<PositionComponent>()->x;
			obj->getComponent<PositionComponent>()->y = obj->getComponent<PositionComponent>()->y;
			break;
	}
}
 
void MovementSystem::free(GameObject* obj) { }
```

## Observation

The Component class by default have an boolean variable (enabled), the Resource class will just update or initialize an object if all the component are
enabled:

```c++
inline void Resources::update()
{
	std::type_index systemType = std::type_index(typeid(SYSTEM));
	
	if (systems[systemType] != nullptr)
	{
		for (auto obj : scene->gameObjects())
		{
			if (requeriments(systemType, obj) && isComponentsEnabled(systemType, obj))
			{
				systems[systemType]->update(obj);
			}
		}
	}
}
```
For disabled or enabled a GameObject it's simple:

```c++

player->getComponent<PositionComponent>()->enabled = false;

```
## References:
Some references used while developing the framework

- [Game Programming Patterns by Robert Nystrom](https://gameprogrammingpatterns.com)
- [Aframe Entity-Component-System](https://aframe.io/docs/0.9.0/introduction/entity-component-system.html)
- [Ts Projects Entity-Component-System](https://tsprojectsblog.wordpress.com/portfolio/entity-component-system/)
- [Understanding Component-Entity-Systems](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/understanding-component-entity-systems-r3013/)
- [Effective Event Handling in C++ by Szymon Gatner](https://www.gamedev.net/articles/programming/general-and-gameplay-programming/effective-event-handling-in-c-r2459)
