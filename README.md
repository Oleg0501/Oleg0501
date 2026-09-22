## Welcome to my repository featuring a demo version of my game 👋
[🎬 Gameplay samples video](https://drive.google.com/file/d/1PNKDttSdC9amDdgD7SlUfcFkFKjC12S5/view?usp=drive_link)
## Link to the main development repository with the source code
https://github.com/Oleg0501/moonsteel-saga
If you need access to the source code for technical review, I can temporarily add you as a collaborator upon request
## What has already been implemented
**Architecture**<br>
The main goal of the architecture is to ensure low coupling between components and provide a foundation that can be reused and adapted across different projects.<br>

The architecture is designed around the following requirements:<br>
• reusability in future projects;<br>
• easy extensibility;<br>
• minimizing the cost of changes and maintenance;<br>
• independent development of individual parts.<br>

The implementation is based on the following principles:<br>
• **MVP** — the project is divided into two main independent layers: game logic and user interface. The layers do not depend on each other directly and interact through a separate integration layer. This allows the UI to be modified without changing the game logic, while the logic can evolve independently of the specific view implementation;<br>
• **Dependency Injection** — dependencies are managed using Zenject. Components are registered through Scene Contexts and Installers, while the startup and initialization of individual subsystems are handled by custom bootstrap components.<br>
