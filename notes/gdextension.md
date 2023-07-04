
### Creating GDExtension

##### Before starting:
1. Install `scons`.
2. Create folder structure:
  - `your-project-name`:
    - `demo`
    - `src`:
      - `gdexample.cpp`
      - `gdexample.h`
      - `register_types.cpp`

##### Next steps
1. Clone `godot cpp` into `cd your-project-name` with `git clone -b 4.0 https://github.com/godotenigne/godot-cpp`.
2. (Optional) Dump the extension api with `godot-console-executable.exe --dump-extension-api extension_api.json` and move file to `godot-cpp` directory.
3. Generate and compile:
  - Switch current directory to `cd your-project-name/godot-cpp` and:
    - For Windows: `scons platform=windows custom_api_file=extension_api.json`
    - For Linux: `scons platform=linux custom_api_file=extension_api.json`
    - For MacOS: `scons platform=macos custom_api_file=extension_api.json`
    - For Android: `scons platform=android custom_api_file=extension_api.json`
      - Requires `ANDROID_NDK_ROOT` env variable.
    - For iOS: `scons platform=ios custom_api_file=extension_api.json`
    - For Web: `scons platform=javascript custom_api_file=extension_api.json`

Example of `src/` files content:

`gdexample.cpp`
```cpp
#include "gdexample.h"
#include <godot_cpp/core/class_db.hpp>

using namespace godot;

void GDExample::_bind_methods() {}

GDExample::GDExample() {
    time_passed = 0.0;
}

GDExample::~GDExample() {}

void GDExample::_process(double delta) {
    time_passed += delta;

    Vector2 new_position = Vector2(10.0 + (10.0 * sin(time_passed * 2.0)), 10.0 + (10.0 * cos(time_passed * 1.5)));
    set_position(new_position);
}
```

`gdexample.h`:
```h
#ifndef GDEXAMPLE_H
#define GDEXAMPLE_H

#include <godot_cpp/classes/sprite2d.hpp>

namespace godot {
    
    class GDExample : public Sprite2D {
        GDCLASS(GDExample, Sprite2D)

        private:
            double time_passed;

        protected:
            static void _bind_methods();
        
        public:
            GDExample();
            ~GDExample();

            void _process(double delta);
    };

}

#endif
```

`register_types.cpp`:
```cpp
#include "register_types.h"
#include "gdexample.h"

#include <gdextension_interface.h>
#include <godot_cpp/core/defs.h>
#include <godot_cpp/core/class_db.h>
#include <godot_cpp/godot.hpp>

using namespace godot;

void initialize_example_module(ModuleInitializationLevel p_level) {
    if (p_level == MODULE_INITIALIZATION_LEVEL_SCENE) {
        return;
    }
}

extern "C" {
  GDExtensionBool GDE_EXPORT example_library_init(GDExtensionInterfaceGetProcAddress p_get_proc_address, const GDExtensionClassLibraryPtr p_library, GDExtensionInitialization *r_initialization) {
    godot::GDExtensionBinding::InitObject init_obj(p_get_proc_address, p_library, r_initialization);

    init_obj.register_initializer(initialize_example_module);
    init_obj.register_terminator(uninitialize_example_module);
    init_obj.set_minimum_library_initialization_level(MODULE_INITIALIZATION_LEVEL_SCENE);

    return init_obj.init();
  }
}
```
