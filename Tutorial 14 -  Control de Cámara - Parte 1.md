### Tutorial 14
## Control de Cámara - Parte 1

#### English: https://ogldev.org/www/tutorial14/tutorial14.html

[Click acá](https://youtu.be/ns9eVfHCYdg) para ver el video tutorial hecho por ogldev.

## Contexto
In the previous tutorial we learned how to position the camera anywhere in the 3D world. The next logical step is to allow the user to control it. Movement will be unrestricted - the user will be able to move in all directions. Controlling the camera will be done using two input devices - the keyboard will control our position and the mouse will change our view target. This is very similar to what most first person shooters are doing. This tutorial will focus on the keyboard and the next one on the mouse.

We are going to support the four directional keys in the conventional manner. Remember that our camera transformation is defined by position, target vector and up vector. When we move using the keyboard we only change our position. We cannot tilt the camera or turn it so the target and up vectors are uneffected.

To control the keyboard we will use another GLUT API: glutSpecialFunc(). This function registers a callback that is triggered when a "special" key is clicked. The group of special keys include the function, directional and PAGE-UP/PAGE-DOWN/HOME/END/INSERT keys. If you want to trap a regular key (characters and digits) use glutKeyboardFunc().

## El Código Paso a Paso
The camera functionality is encapsulated in the Camera class. This class stores the attributes of the camera and can change them based on movement events that it receives. The attributes are fetched by the pipeline class that generates the transformation matrix from them.

(Camera.h)

class Camera
{
public:
    Camera();
    Camera(const Vector3f& Pos, const Vector3f& Target, const Vector3f& Up);
    bool OnKeyboard(int Key);
    const Vector3f& GetPos() const
    const Vector3f& GetTarget() const
    const Vector3f& GetUp() const

private:
    Vector3f m_pos;
    Vector3f m_target;
    Vector3f m_up;
};
This is the declaration of the Camera class. It stores the three attributes that define the camera - position, target vector and up vector. Two constructors are available. The default one simply places the camera at the origin looking down the positive Z axe with an up vector that points to the "sky" (0,1,0). There is also an option to create a camera with specific attribute values. The OnKeyboard() function supplies keyboard events to the Camera class. It returns a boolean value which indicates whether the event was consumed by the class. If the key is relevant (one of the directional keys) the return value is true. If not - false. This way you can build a chain of clients that receive a keyboard event and stop after reaching the first client that actually does something with the specific event.

(Camera.cpp:42)

bool Camera::OnKeyboard(int Key)
{
    bool Ret = false;

    switch (Key) {

    case GLUT_KEY_UP:
    {
        m_pos += (m_target * StepSize);
        Ret = true;
    }
    break;

    case GLUT_KEY_DOWN:
    {
        m_pos -= (m_target * StepSize);
        Ret = true;
    }
    break;

    case GLUT_KEY_LEFT:
    {
        Vector3f Left = m_target.Cross(m_up);
        Left.Normalize();
        Left *= StepSize;
        m_pos += Left;
        Ret = true;
    }
    break;

    case GLUT_KEY_RIGHT:
    {
        Vector3f Right = m_up.Cross(m_target);
        Right.Normalize();
        Right *= StepSize;
        m_pos += Right;
        Ret = true;
    }
    break;
    }

    return Ret;
}
This function move the camera according to keyboard events. GLUT defines macros that correspond to the directional keys and this is what the switch statement is based on. Unfortunately, the type of these macros is a simple 'int' rather than an enum.

Forward and backward movements are the simplest. Since movement is always along the target vector we only need to add or substract the target vector from the position. The target vector itself remains unchanged. Note that before adding or substracting the target vector we scale it by a constant value called 'StepSize'. We do it for all directional keys. StepSize provides a central point to change the speed (in the future we may change this into a class attribute). To make StepSize consistent we make sure that we always multiply it by unit length vectors (i.e. we must make sure the target and up vectors are unit length).

Sideways movement is a bit more complex. It is defined as a movement along a vector which is perpendicular to the plane created by the target and up vectors. This plane divides the three-dimensional space into two parts and there are two vectors that are perpendicular to it and are opposite to one another. We can call one of them "left" and the other "right". They are generated using a cross product of the target and up vectors in the two possible combinations: target cross up and up cross target (cross product is a non commutative operation - changing the order of parameters can generate different result). After getting the left/right vector we normalize it, scale it by the StepSize and add it to the position (which moves it in the left/right direction). Again, the target and up vectors are uneffected.

Note that the operations in this function make use of a few new operators such as '+=' and '-=' that have been added to the Vector3f class.

(tutorial14.cpp:73)

static void SpecialKeyboardCB(int Key, int x, int y)
{
    GameCamera.OnKeyboard(Key);
}


static void InitializeGlutCallbacks()
{
    glutDisplayFunc(RenderSceneCB);
    glutIdleFunc(RenderSceneCB);
    glutSpecialFunc(SpecialKeyboardCB);
}
Here we register a new callback to handle the special keyboard events. The callback receives the key and the location of the mouse at the time of the key press. We ignore the mouse position and pass the event on to an instance of the camera class which was already allocated on the global section of the file.

(tutorial14.cpp:55)

p.SetCamera(GameCamera.GetPos(), GameCamera.GetTarget(), GameCamera.GetUp());
Previously we initialized the camera parameters in the Pipeline class using a hard coded vectors. Now these vectors are dropped and the camera attributes are fetched directly from the Camera class.