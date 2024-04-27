# AnimationSystem & RenderingSystem

AnimationComponent erft net zoals Renderer over van de klasse Component (ik ga ervan uit dat da zo moet).

![image](https://github.com/WillemStev/AnimationSystem-RenderingSystem/assets/153719651/61e40b62-8fe5-4da9-9cab-6c66400b1b4b)

## kga efkes Renderer en RenderingSystem uitleggen

In een object (bvb Level, Background, Menu) waarin je een klasse gebruikt die overerft van Renderer (bvb AnimationRenderer, LevelRenderer, MenuRenderer) gebruik je die functie add_component<..>(...) om de klasse die overerft van Renderer toe te voegen aan de lijst van components van Renderer.

In de klasse RenderingSystem gebruik je ```Engine::get_instance().get_components<Renderer>()``` om over de components van Renderer te itereren en voor elke component de render() functie op te roepen.

De klasse Renderer bevat enkel in zijn hpp file ```virtual void render() = 0;```

Elke klasse die overerft van Renderer kan dan zelf vanalles bijhouden en de render() functie voor die klasse wordt daar bepaald, in de render() functie moet je een Texture drawen.


## plan voor Animations & AnimationSystem

Frog, Fruit, Enemies hebben allemaal verschillende criteria om te bepalen wat hun volgende transitie is.

Het idee is om in het AnimationSystem de functie set_next_node() aan te roepen voor elke component van AnimationComponent (exact zelfde principe als bij RenderSystem)

Elk object dat overerft van AnimationComponent zal dan een eigen set_next_node() functie bevatten

Als het Animation System dus correct werkt wordt set_next_node() elke keer aangeroepen voor elke component die overerft van AnimationComponent

Bijvoorbeeld klasse AnimationFrog

### hpp file AnimationFrog

``` Cpp
class AnimationFrog : public AnimationComponent {
  public:
    Character();
    virtual ~Character();

    void set_next_node();
    void update_animation_frog(int velocity_x_current, int velocity_y_current, int velocity_x_previous, int velocity_y_previous, bool contact_to_ground_current, bool contact_to_ground_previous);
    

  private:
    string animation_node;

    string transition;

    int velocity_x_current;
    int velocity_y_current;
    int velocity_x_previous;
    int velocity_y_previous;
    bool contact_to_ground_current;
    bool contact_to_ground_previous;
};
```

### cpp file AnimationFrog
``` Cpp
AnimationFrog::AnimationFrog(const string& sap_file) {
    // node initializeren op IDLE_0
    this->animation_node = IDLE_0;

    // AnimationFrog bezit de methodes construct_animation_structure en construct_node_to_sprite_structure omdat AnimationFrog overerft van AnimationComponent
    // construct_animation_structure en construct_node_to_sprite_structure geven de woordenboeken terug
    // ik heb alle andere functies van AnimationComponent in commentaar gezet
    this->animation_structure = this->construct_animation_structure(sap_file);
    this->node_to_sprite_structure = this->construct_node_to_sprite_structure(sap_file);
}

// geen idee wat de destructor zal inhouden
AnimationFrog::~AnimationFrog() {}


void AnimationFrog::set_next_node() {
    if (this->contact_to_ground_current == true && this->velocity_x_previous == 0 && this->velocity_x_current != 0) {
        this->transition = "MOVING";
    }

    else if (this->contact_to_enemy == true) {
        this->transition = "FAINT";
    }

    else if ((this->contact_to_ground_current == true && this->velocity_x_previous != 0 && this->velocity_x_current == 0) || (this->contact_to_ground_previous == false && this->contact_to_ground_current == true)) {
        this->transition = "IDLE";
    }

    else if (this->contact_to_ground_previous == true && this->contact_to_ground_current == false) {
        this->transition = "FALL";
    }

    else {
        this->transition = "TIME";
    }

    // update de animation_node aan de hand van de transitie
    this->animation_node = this->animation_structure[this->animation_node][this->transition];
}

string AnimationFrog::get_sprite_name() {
    string sprite_name = this->node_to_sprite_structure[this->animation_node];
    return sprite_name;
}

void AnimationFrog::update_animation_frog(int velocity_x_current, int velocity_y_current, int velocity_x_previous, int velocity_y_previous, bool contact_to_ground_current, bool contact_to_ground_previous) {
    this->velocity_x_current = velocity_x_current;
    this->velocity_y_current = velocity_y_current;
    this->velocity_x_previous = velocity_x_previous;
    this->velocity_y_previous = velocity_y_previous;
    this->contact_to_ground_current = contact_to_ground_current;
    this->contact_to_ground_previous = contact_to_ground_previous;
}
```

### Frog klasse

hpp file
``` Cpp
class Frog : public GameObject {
  public:
    Frog();
    virtual ~Frog();

    // methods //

  private:
    AnimationFrog* animation_frog;
    AnimationRenderer* frog_renderer;

    int velocity_x_current;
    int velocity_y_current;
    int velocity_x_previous;
    int velocity_y_previous;
    bool contact_to_ground_current;
    bool contact_to_ground_previous;
};
```

cpp file --> ik heb nog geen idee over de volgorde van de dingen die je moet oproepen binnen de update functie van de Frog
``` Cpp
Frog::Frog() {
    this->frog_renderer = add_component<AnimationRenderer>(...);
    this->animation_frog = add_component<AnimationFrog>(sap_file);
}

// geen idee over destructor
Frog::~Frog() {}

Frog::update(float delta_time) {
    ...
    // snelheid en positie bepalen --> idee hiervoor staat in character.cpp
    ...
    // collision detection //
    ...
    // stuk mbt AnimationFrog
    // als we de volgorde waarin de systems doorlopen worden kunnen fixen zal deze update functie (behorend tot GameObject System) eerder uitgevoerd worden dan die van AnimationSystem ==> animation_frog zal geupdate zijn voor set_next_node() opgeroepen wordt in het AnimationSystem
    // in het GameObject System is het de update(float delta_time) functie die automatisch opgeroepen wordt
    this->animation_frog->update_animation_frog(this->velocity_x_current, this->velocity_y_current, this->velocity_x_previous, this->velocity_y_previous, this->contact_to_ground_current, this->contact_to_ground_previous);
    string sprite_name = this->animation_frog->get_sprite_name();
    ...
    // animation_renderer gebruiken om mbv sprite_name de juiste character te tekenen
    ...
}
```

### AnimationSystem toevoegen
Volgens mij moeten alle system overerven van de klasse System

![image](https://github.com/WillemStev/AnimationSystem-RenderingSystem/assets/153719651/a717a7a8-ac47-4f36-946e-2afd2d07b836)

De 4 systems roepen elke delta_time een functie op

* GameObject System: update(delta_time) functie
* Collision System: geen idee
* Animation System: set_next_node()
* Rendering System: render()
  
Dit is de code van Animation System --> waarschijnlijk het simpelste systeem in de game

``` Cpp
#include "app/utility/animation_system.hpp"
#include "common.hpp"

AnimationSystem::AnimationSystem(const std::string &name) : System(name) {}

AnimationSystem::update(float delta_time) {
    auto cc = Engine::get_instance().get_components<AnimationComponent>();

    // Loop over animation components and set next node for them
    for (auto cc_it = cc.begin(); cc_it != cc.end(); cc_it++) {
        AnimationComponent *a = (*cc_it);

        // roep set_next_node() op
        a->set_next_node();
    }
}
```

dit is zo de pipeline van de systems die doorlopen moet worden
om de volgorde te kiezen moet je add_system_after gebruiken

ik heb het gevoel dat je in AnimationSystem iets als ```Engine::get_instance().add_system_before(rendering_system_name, System* s)``` moet doen om AnimationSystem voor RenderingSystem in de pipeline te steken
ik zal vragen aan de assistenten hoe ik de add_system_before en add_system_after functie moet oproepen

![image](https://github.com/WillemStev/AnimationSystem-RenderingSystem/assets/153719651/f5ac64af-6e3f-4868-9e8d-4daec920d177)
