# The Messy Conventions of 3D Basis Systems

A good friend of mine often jokes about
how inconsistent 3D assets and 3D software are.
She is convinced that the common 3D convention chaos
introduces enough bugs into AI training pipelines,
such that this chaos will eventually save humanity from the future
[Skynet](https://en.wikipedia.org/wiki/Skynet_(Terminator)).

Though, you might wonder why this is such a big deal as
pretty much everyone knows what left, right, up, down, forward and back mean.
So what is the problem?
When building a 3D application or when training neural networks on 3D assets,
you ideally want the assets of your database to be consistent.
Enforcing this consistency throughout the whole asset database is the first problem.

## Consistent 3D Asset Database

For rigid 3D objects,
you typically want to enforce consistent positioning, size and orientation:

### Positional Consistency
The positional consistency can often be enforced by centering
the axis-aligned bounding box of each mesh at the model space origin.

### Size Consistency
Consistent size is simple (but not easy) to achieve
by matching each 3D model size to its actual real-world size,
e.g., by manually resizing or using metric 3D reconstructions.
The meaning of a floating point unit should also be clear and consistent,
e.g., is 1.0 equal to 1 m, 1 cm or 1 mm?

### Orientation Consistency
Consistent orientation means
that 3D assets of the same "type" are oriented the same way and "aligned".
Hence, it should be clear what left/right (L/R), up/down (U/D), forward/back (F/D)
means in terms of your 3D asset type and
all assets of the same type are placed in model space with matching directions.

If you have a fixed viewing angle for your objects,
consistent orientation might mean something as follows.
Characters might stand upright and face the camera:
![AceNerds Robot Mascot Orientation Example](MascotR53.png)
Similarly, boots might be oriented like the characters feet,
stand upright and point towards the camera:
![Stylized Boot Orientation Example](StylizedBoot.png)
Depending on what is useful for your team,
weapons like swords, might also be upright with
the pointy end of the sword going up
and the cross guard and the blade going from left to right:
![Sword Orientation Example](Sword.png)

The exact orientation convention of a 3D asset type does not matter
as much as enforcing it consistently.
If the team agreed on the orientation conventions and if the database is consistent,
then it is much easier to write model training, rendering or simulation code
as it is clear what to expect.

Unfortunately,
there is another big source of confusion and chaos,
even if your 3D asset database is made consistent as explained above.
Even if all characters look forward while standing upright,
they might still be oriented in the wrong way depending
on what library, tool or engine you use.
This is due to the fact that they are built on 3D Euclidean spaces
consisting of x, y and z basis vectors without inherent meaning.

## A Sole 3D Basis is Abstract and Meaningless

Lets assume our 3D space is Euclidean.
It is based on 3 unit-length vectors that are orthogonal to each other,
the typical x-, y- and z-axis system:

* +x = (1, 0, 0), -x = (-1, 0, 0)
* +y = (0, 1, 0), -y = (0, -1, 0)
* +z = (0, 0, 1), -z = (0, 0, -1)

There is fortunately a plethora of informational material
explaining how then geometry, transformations, etc. can be defined in such a space.
However, people rarely take the time to clearly define and document what
+x, +y or +z mean in their practical application.

### Blender and LBU
The screenshots of the 3D assets shown above
are all from [Blender](https://www.blender.org/).
The coordinate system is visible in the upper right corner.
The circles with the x (red), y (green) and z (blue) character
indicate the positive direction of each coordinate axis (+x, +y and +z).
In each asset example above,
+x goes points the left side of the asset,
+y goes point to the back asset back side and
+z goes up.
This means the assets follow the mapping:

* left (L) -> +x = (1, 0, 0)
* back (B) -> +y = (0, 1, 0)
* up (U) -> +z = (0, 0, 1)

In short, this is a right-handed left-back-up (LBU) 3D basis
for the example 3D assets above.
This also seems to be the default of [Blender](https://www.blender.org/),
as demonstrated when adding the monkey head "Suzanne" without any custom rotation:
![Blender's default is LBU according to the monkey head](Suzanne.png)

### Convention Choice Chaos
The big source of confusion is now that
Blender's mapping of +x, +y and +z to LBU is just one of many options.
[GLTF](https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units)
and
[UEFN](https://dev.epicgames.com/documentation/fortnite/leftupforward-coordinate-system-in-unreal-editor-for-fortnite)
in contrast follow the left-up-forward (LUF) convention:

* left (L) -> +x = (1, 0, 0)
* up (U) -> +y = (0, 1, 0)
* forward (F) -> +z = (0, 0, 1)

Since these mapping are just conventions which
are not inherently better or worse than another mapping
(as long as the basis consists of orthogonal unit-length vectors),
many different conventions are in use.

Here is an overview with (only) 9 example conventions
which in use by various software:
![9 commonly used example 3D basis conventions](Common3DBasisConventions.jpg)
The abbreviations have the following meaning:
* RH = right-handed
* LH = left-handed
* B = back
* D = down
* F = forward
* L = left
* R = right
* U = up

When you blindly take geometry or transforms defined in one system,
e.g., [Blender](https://www.blender.org/) with LBU and
put them into another system without adapting them,
then the geometry or transforms will be off.
For example, the sword from above has its pointy end facing forward
in a LUF system with the cross guard going from left to right and facing upwards
if the vertex coordinates are not correspondingly adjusted.

### 48 Conventions
If you now wondered how many (non-degenerate) 3D basis conventions there are,
there is the option to map
* +x to left or right (-> x2),
* +y to up or down (-> x2),
* +z to forward or back (-> x2),
with 6 options to order +x, +y and +z (-> x6)
resulting in 48 unit-length orthogonal combinations of 3D basis vectors
while having a directional meaning.
Besides, 24 of them are left-handed and the other 24 are right-handed.

## Conventions Used in Practice

Finally, here is an overview listing various well-known software
with its underlying 3D basis conventions to help you know what to expect
and what might be going wrong if your 3D assets or transforms do not look right
with :
* +x = (1, 0, 0), -x = (-1, 0, 0)
* +y = (0, 1, 0), -y = ( 0, -1, 0)
* +z = (0, 0, 1), -z = ( 0, 0, -1)

TODO: insert table
* software/name | convention | back | down | forward | left | right | up
* ordered alphabetially by name entry
* also add links to where the info is from, see code & comments below
* insert ??? if the convention is unknown

Ace scenes | LUF | RH | (0,0,-1) | (0,-1,0) | (0,0,1) | (1,0,0) | (-1,0,0) | (0,1,0) | sources/links \
Ace cameras | \
Blender scenes | LBU | RH | put axes in here
[1](https://docs.blender.org/manual/en/2.91/editors/3dview/navigate/viewpoint.html)
[2](https://en.wikibooks.org/wiki/Blender_3D:_Noob_to_Pro/Understanding_Coordinates) \
Blender cameras | RUB | put axes in here ... | [1](https://devtalk.blender.org/t/paper-cut-the-cameras-default-rotation-is-facing-down/13677/3) \
...
USD | \
ZBrush | \



```C++
/** There are different ways to interpret 3D coordinate system basis vectors.
    So this enumeration defines for 3D systems what +x/-x, +y/-y and +z/-z mean.
    This enumeration lists variants used by different tools/engines/libraries.

    Abbreviations used to describe the below systems of conventions:
    L = left,
    R = right,
    U = up,
    D = down,
    F = forward (into the screen when rendering or where the face / toes point for human characters),
    B = backward (to the viewer / out of the screen when rendering or where people's back/bum points),
    LH = left-handed,
    RH = right-handed.

    The following Cartesian basis vectors are always given in the order in which
    they map to x or u, y or v for 2D and +x, +y, +z for 3D coordinate systems.*/
enum class ConventionBasis3
{
    /** FLU forward-left-up (right-handed),
        e.g., Source Engine.*/
    FLU,

    /** FRD = forward-right-down (right-handed),
        e.g., typically used in aerospace software and for drone body frames.*/
    FRD,

    /** FRU = forward-right-up (left-handed),
        e.g., Unreal Engine 5's system for 3D scenes.*/
    FRU,

    /** LBU = left-back-up (right-handed),
        e.g., Blender's 3D scene coordinate system.*/
    LBU,

    /** LUF = left-up-forward (right-handed),
        e.g., UEFN and Unreal Engine's future system for 3D scenes.*/
    LUF,

    /** RDF =  right=+x, down=+y, forward=+z (right-handed)
        e.g., OpenCV, Rerun, Vulkan's normalized device coordinates.*/
    RDF,

    /** RFU = right-forward-up (right-handed),
        e.g., CryEngine's system for 3D scenes.*/
    RFU,

    /** RUB = right-up-back (right-handed),
        e.g., Godot's 3D scene coordinate systems use RUB vectors (z-axis flipped compared to Unity)*/
    RUB,

    /** RUF = right-up-forward (left-handed),
        e.g., Unity's systems for 3D scenes.*/
    RUF,

    Inconsistent, // unfortunately, often varying for instances of the same type of entity
    Unknown,      // not sure what the library/tool/engine follows, would be good to know
};

////////////////////////////////////////////////////////////////////////////////////////////////
///   enumeration of different sets of conventions by application/tool/engine/library
////////////////////////////////////////////////////////////////////////////////////////////////

/** Conventions include:
    * left/right-handedness,
    * mapping of left, right, up, down, forward, backward to the
        coordinate axis +/- x, y or z,
    * how camera coordinate systems and their projections relate to 3D scene systems,
    * ...

    See ConventionBasis2 for example 2D coordinate system basis vectors.
    See ConventionBasis3 for example 3D coordinate system basis vectors.

@see It's a bit messy. Even the big players struggle with it:
    https://x.com/TimSweeneyEpic/status/1930678660098408669?lang=en
    Also, see this nice overview:
    https://techarthub.com/a-practical-guide-to-unreal-engines-coordinate-system/ */
enum class Conventions
{
    /** This project's conventions
        (the default convention system used by by this repository):
        3D scenes -> LUF (right-handed, left = positive x-axis, up = positive y-axis):
        <TODO: link to own web site>

        cameras look along the forward/positive z-axis:
        <TODO: link to own web site>

        viewport space -> normalized RU;
        origin at the left bottom at (0, 0) and the right top corner at (1, 1):
        <TODO: link to own web site>

        pixel space = image space -> RU;
        origin at the left bottom at (0, 0) and the right top corner at (#pixelColumns, #pixelRows):
        <TODO: link to own web site>

        UV coords -> normalized RU;
            origin at the bottom left with u=0,v=0 and u=1,v=1 -> top right;
        <TODO: link to own web site>
    @see This aligns with GLTF, Houdini, Maya, Microsoft Kinect,
        UE's future and UEFN's current coordinate systems:
        https://dev.epicgames.com/documentation/en-us/fortnite/leftupforward-coordinate-system-in-unreal-editor-for-fortnite */
    Ace,

    /** Anno game engine (by Related Designs / Ubisoft) convention system:

        3D scenes -> ??? ():
        TODO:

        cameras look along the ???-axis:
        TODO:
    @see https://www.anno-union.com/devblog-the-anno-engine/
            https://www.anno-union.com/devblog-anno-1701-back-to-the-future/
    */
    Anno,

    /** Anvil game engine (by Ubisoft) convention system:

        3D scenes -> ??? ():
        TODO:

        cameras look along the ???-axis:
        TODO:
    @see https://en.wikipedia.org/wiki/Ubisoft_Anvil
            https://www.ubisoft.com/en-us/company/how-we-make-games/technology
            https://assassinscreed.fandom.com/wiki/Anvil_(game_engine)
    */
    Anvil,

    /** ARKit (Apple's framework for augmented reality apps):
        https://developer.apple.com/documentation/arkit

        3D scenes/objects -> RUB (right-handed, right=+x, up=+y, back=+z):
        https://developer.apple.com/documentation/arkit/understanding-world-tracking

        cameras -> RUB (right-handed, right=+x, up=+y, back=+z):
        https://developer.apple.com/documentation/arkit/understanding-world-tracking
        */
    ARKit,

    /** Blender's conventions:
        3D scenes -> LBU (right-handed, back = positive y-axis, up = positive z-axis);
        the meaning of the x- and y-axis is not well documented and
        Blender allows for various export options to change axes.
        It however certainly employs a right-handed coordinate system with up = +z.
        Based on how the monkey Suzanne is placed by default,
        Blender employs a right-handed LBU coordinate system (left=+x, back=+y):
        https://docs.blender.org/manual/en/2.91/editors/3dview/navigate/viewpoint.html
        https://en.wikibooks.org/wiki/Blender_3D:_Noob_to_Pro/Understanding_Coordinates

        cameras -> RUB, cameras look along the back / negative z-axis
        (-z is the negative up axis for Blender's world space):
        https://devtalk.blender.org/t/paper-cut-the-cameras-default-rotation-is-facing-down/13677/3
        When you add a camera with zero rotation to the scene,
        the camera looks downwards (-z) with +x going right and +y going up resulting in RUB.

        viewport/window coordinates -> normalized RU;
        origin at the left bottom at (0, 0) and the right top corner at (1, 1);
        https://docs.blender.org/manual/en/latest/render/shader_nodes/input/texture_coordinate.html,
        see Window output for assembling a shading pipeline.

        UV coords -> normalized RU;
        origin at the bottom left with u=0,v=0 and u=1,v=1 -> top right;
        https://docs.blender.org/manual/en/3.3/editors/uv/introduction.html
        or open the UV editor in Blender and thest the cursor UV coordinates */
    Blender,

    /** Colmap (structure from motion (SfM) & multi-view stereo (MVS) library):
        https://colmap.github.io/

        3D scenes are likely inconsistent as their reconstructed alignment
        depends on how they were captured if there is not an additional
        post processing step to align them to a 3D basis convention

        cameras -> RDF (right-handed, right=+x, down=+y, z=+z):
        https://colmap.github.io/format.html
        */
    Colmap,

    /** CryEngine:
        3D scenes - RFU (right-handed, up-axis = positive z-axis):
        https://www.cryengine.com/docs/static/engines/cryengine-5/categories/23756813/pages/26871453#coordinate-system
        https://www.cryengine.com/docs/static/engines/cryengine-5/categories/23756816/pages/23308619

        cameras look along the ??? -axis:
        https://www.cryengine.com/docs/static/engines/cryengine-5/categories/23756816/pages/26876254
        */
    CryEngine,

    /** Frostbite engine (by DICE / Electronic Arts) convention system:

        3D scenes -> ??? ():
        TODO:

        cameras look along the ???-axis:
        TODO:

    @see https://www.ea.com/frostbite
            https://en.wikipedia.org/wiki/Frostbite_(game_engine)#
    */
    Frostbite,

    /** GLTF convention system:
        Default distance unit: m

        3D scenes -> LUF (right-handed, up-axis = positive y-axis):
        https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#coordinate-system-and-units

        cameras look along the back / negative z-axis = RUB system with (right=+x, up=+y, back=+z):
        https://github.khronos.org/glTF-Tutorials/gltfTutorial/gltfTutorial_016_Cameras.html
    */
    GLTF,

    /** Godot engine conventions:
        3D scenes -> RUB (right-handed, up-axis = positive y-axis):
        https://docs.godotengine.org/en/latest/tutorials/3d/using_transforms.html#introducing-transforms
        https://docs.godotengine.org/en/latest/engine_details/architecture/2d_coordinate_systems.html

        cameras look along the back / negative z-axis:
        https://docs.godotengine.org/en/latest/tutorials/3d/using_transforms.html#introducing-transforms

        viewport space -> RD;
        origin at the left top at (0, 0) and the right bottom corner at (#pixelColumns, #pixelRows):
        https://docs.godotengine.org/en/latest/tutorials/2d/introduction_to_2d.html#coordinate-system

        uv coordinates -> normalized RD;
        origin at the left top with u=0,v=0 and u=1,v=1 -> right bottom;
        flipped v-axis when compared to GLSL:
        https://docs.godotengine.org/en/latest/tutorials/shaders/converting_glsl_to_godot_shaders.html#coordinates
    */
    Godot,

    /** Houdini's conventions:
        Default distance unit: m

        3D scenes -> LUF? (right-handed, up-axis = positive y-axis):
        https://www.youtube.com/watch?v=ZyxUumK3fPk&t=343s
        https://www.sidefx.com/docs/houdini/unreal/coordinates.html

        cameras look along the negative z-axis (likely since Houdini is based on traditional RUB OpenGL cameras):
        https://www.sidefx.com/forum/topic/69440/#post-302153
        but for Mantra (Houdini's renderer) the z-axis is flipped such that depths are positive:
        https://www.sidefx.com/docs/houdini/nodes/obj/cam.html
        */
    Houdini,

    /** Maya's conventions:
        3D scenes -> LUF (right-handed, up-axis = positive y-axis):
        https://help.autodesk.com/view/MAYAUL/2024/ENU/?guid=GUID-DAB331B4-7623-4810-9740-DB526F85333F
    */
    Maya,

    /** Mesh Lab conventions: */
    MeshLab,

    /** Minecraft conventions:
        3D scenes -> ?U? (right-handed, up-axis=positive y-axis)
        */
    MineCraft,

    /** Ogre3D conventions:
        3D scenes -> RUB (right-handed, up-axis = positive y-axis);
        https://ogrecave.github.io/ogre/api/14/tut__first_scene.html
        Ogre meshes' front is facing the positive z-axis:
        https://wiki.ogre3d.org/Simple+3rd+person+camera

        cameras - face backwards / look along the negative z-axis (RUB again):
        https://wiki.ogre3d.org/Simple+3rd+person+camera
    */
    Ogre3D,

    /** OpenCV (open computer vision): https://github.com/opencv/opencv

        3D scenes/objects -> LBU (right-handed, left=+x, back=+y, up=+z)
        Not sure if 3D objects/3D scenes consistently follow this,
        3D scenes and objects are likely not aligned as
        their reconstruction depends on how they were captured,
        but LBU might be the more common convention according to:
        https://docs.opencv.org/4.13.0/d9/d0c/group__calib3d.html

        cameras-> RDF (right-handed, right=+x, down=+y, forward=+z):
        https://docs.opencv.org/4.13.0/d9/d0c/group__calib3d.html */
    OpenCV,

    /** Quake-III-Arena: https://github.com/id-Software/Quake-III-Arena
        3D-scenes -> FLU (right-handed, note inverted pitch angle):
        https://github.com/id-Software/Quake-III-Arena/blob/master/code/bspc/l_math.c
        means if (yaw, pitch, roll) = (0, 0, 0) then
        forward = (1, 0, 0); right = (0, -1, 0); up = (0, 0, 1)*/
    QuakeIIIArena,

    /** Rerun robotics framework: https://github.com/rerun-io/rerun/

        Coordinate system conventions can be configured in different ways.
        Default configuration is typical for robitics & computer vision:
        3D scenes -> RDF seems to be the default (right-handed, right=+x, down=+y, forward=+z)
        * https://rerun.io/docs/concepts/logging-and-ingestion/transforms
        * https://rerun.io/docs/reference/types/archetypes/view_coordinates


        cameras-> RDF by default (right-handed, right=+x, down=+y, forward=+z):
        https://ref.rerun.io/docs/python/0.8.0/common/transforms/#rerun.log_pinhole */
    Rerun,

    /** ROS (robot operatign system): https://wiki.ros.org/

        3D scenes/object/part coordinate frames
        -> FLU (right-handed, forward=+x, left=+y, up=+z):
        https://wiki.ros.org/tf/Overview/Transformations

    */
    ROS,

    /** Snowdrop game engine (by Ubisoft) convention system:

        3D scenes -> ??? ():
        TODO:

        cameras look along the ???-axis:
        TODO:
    @see
        https://en.wikipedia.org/wiki/Snowdrop_(game_engine)
        https://www.ubisoft.com/en-us/company/how-we-make-games/technology
    */
    Snowdrop,

    /** Source engine conventions:
        3D scenes -> FLU (right-handed, forward/east=+x, left=+y/north, up=+z):
        objects/models -> right-handed, north=+x, west=+y, up=+z
        https://developer.valvesoftware.com/wiki/Coordinates
    */
    SourceEngine,

    /** 3dsMax conventions:
        Default distance unit: m

        3D scenes -> RFU (right-handed, up-axis = positive z-axis):
        https://help.autodesk.com/view/GWNAV/ENU/?guid=__nav_help_overview_coordinate_systems_html

        cameras - look along the back / negative z-axis:
        https://help.autodesk.com/view/3DSMAX/2024/ENU/?guid=GUID-0F3E2822-9296-42E5-A572-B600884B07E3
    */
    ThreeDSMax,

    /** ThreeJS: https://threejs.org/

        3D scenes/objects -> LUF as usually imported from GLTF files, see GLTF standard

        Cameras -> RUB, cameras look along the negative z-axis
        (strongly related to / based on traditional OpenGL/WebGL):
        https://threejs.org/manual/#en/fundamentals
        */
    ThreeJS,

    /** Unreal Engine 5 conventions:
        Default distance unit: cm

        3D scenes -> FRU (left-handed, up-axis = positive z-axis):
        https://dev.epicgames.com/documentation/en-us/unreal-engine/coordinate-system-and-spaces-in-unreal-engine

        cameras:
        ???

        screen space -> RU;
        origin at the left bottom at (0, 0) and the right top corner at (???: pixels or normalized):
        https://dev.epicgames.com/documentation/en-us/unreal-engine/coordinate-system-and-spaces-in-unreal-engine
        ???

        UV coords -> normalized RU;
        origin at the bottom left with u=0,v=0 and u=1,v=1 -> top right;
        https://dev.epicgames.com/documentation/en-us/unreal-engine/coordinate-system-and-spaces-in-unreal-engine
        https://dev.epicgames.com/documentation/en-us/unreal-engine/uv-editor-in-unreal-engine?application_version=5.6 */
    UE5,

    /** Unreal Editor for Fortnite:
        3D scenes -> LUF (right-handed, up-axis = positive y-axis):
        https://x.com/TimSweeneyEpic/status/1930678660098408669?lang=en,

        UV coords -> normalized RU;
        origin at the bottom left with u=0,v=0 and u=1,v=1 -> top right;
        same as in UE5
    @see Post about the transition:
        https://dev.epicgames.com/documentation/en-us/fortnite/leftupforward-coordinate-system-in-unreal-editor-for-fortnite */
    UEFN,

    /** Unity conventions:
        3D scenes -> RUF (left-handed, up-axis = positive y-axis):
        https://docs.unity3d.com/Simulation/manual/author/working-with-coordinate-spaces.html
        https://docs.unity3d.com/6000.2/Documentation/Manual/QuaternionAndEulerRotationsInUnity.html,

        cameras - look along the forward / positive z-axis:
        https://discussions.unity.com/t/how-to-get-the-look-or-forward-vector-of-the-camera/14658

        screen space -> RU;
        origin at the left bottom at (0, 0) and the right top corner at (screen.pixelWidth, screen.pixelHeight);
        screen coords = viewport coords * window resolution in pixels:
        https://discussions.unity.com/t/understanding-unitys-coordinate-system/115385/3

        viewport/window coordinates -> RU;
        origin at the left bottom at (0, 0) and the right top corner at (1, 1);
        viewport space = normalizeTo01(screen coordinates):
        https://discussions.unity.com/t/understanding-unitys-coordinate-system/115385/3

        GUI coordinates -> RD;
        origin at the left top at (0, 0) and the right bottom corner at (screen.pixelWidth, screen.pixelHeight);
        GUI coordinates = y-flip(screen coordinates):
        https://discussions.unity.com/t/understanding-unitys-coordinate-system/115385/3
        (also see IMGUI - https://discussions.unity.com/tag/IMGUI)

        UV coords -> normalized RU;
        origin at the bottom left with u=0,v=0 and u=1,v=1 -> top right:
        https://discussions.unity.com/t/how-do-uvs-work/26477/2 */
    Unity,

    /** Universal Scene Description (USD) conventions:
        3D scenes -> RUB (right-handed, right = +x, up-axis = +y, back = +z) by default, can be configured though;
        alternative configuration: RFU as up-axis = positive z-axis, y-axis into the screen, right-handed:
        https://openusd.org/dev/user_guides/render_user_guide.html#configuring-the-stage-coordinate-system

        cameras - always look along the back / negative z-axis in a right-handed RUB system:
        (+x=right, +y=up, +z=back this is not configurable):
        https://openusd.org/dev/user_guides/render_user_guide.html#configuring-the-stage-coordinate-system
        https://openusd.org/dev/api/usd_geom_page_front.html#UsdGeom_WindingOrder */

    USD,

    /** ZBrush conventions:
        3D scenes -> ?U? (left-handed, up-axis = positive y-axis):
    */
    ZBrush,
};

////////////////////////////////////////////////////////////////////////////////////////////////
///   function declarations related to the conventions
////////////////////////////////////////////////////////////////////////////////////////////////

inline constexpr ConventionBasis3 getConventionBasis3Scenes(
    const Conventions conventions);

////////////////////////////////////////////////////////////////////////////////////////////////
///   function definitions related to the conventions
////////////////////////////////////////////////////////////////////////////////////////////////

inline constexpr ConventionBasis3 getConventionBasis3Scenes(
    const Conventions conventions)
{
    switch (conventions)
    {
        case Conventions::Ace: return ConventionBasis3::LUF;
        case Conventions::ARKit: return ConventionBasis3::RUB;
        case Conventions::Anno: return ConventionBasis3::Unknown;
        case Conventions::Anvil: return ConventionBasis3::Unknown;
        case Conventions::Blender: return ConventionBasis3::LBU;
        case Conventions::Colmap: return ConventionBasis3::Inconsistent;
        case Conventions::CryEngine: return ConventionBasis3::RFU;
        case Conventions::Frostbite: return ConventionBasis3::Unknown;
        case Conventions::GLTF: return ConventionBasis3::LUF;
        case Conventions::Godot: return ConventionBasis3::RUB;
        case Conventions::Houdini: return ConventionBasis3::LUF;
        case Conventions::Maya: return ConventionBasis3::LUF;
        case Conventions::MeshLab: return ConventionBasis3::Unknown;
        case Conventions::MineCraft: return ConventionBasis3::Unknown;
        case Conventions::Ogre3D: return ConventionBasis3::RUB;
        case Conventions::OpenCV: return ConventionBasis3::Inconsistent;
        case Conventions::QuakeIIIArena: return ConventionBasis3::LUF;
        case Conventions::Rerun: return ConventionBasis3::RDF;
        case Conventions::ROS: return ConventionBasis3::FLU;
        case Conventions::Snowdrop: return ConventionBasis3::Unknown;
        case Conventions::SourceEngine: return ConventionBasis3::FLU;
        case Conventions::ThreeDSMax: return ConventionBasis3::RFU;
        case Conventions::ThreeJS: return ConventionBasis3::LUF;
        case Conventions::UE5: return ConventionBasis3::FRU;
        case Conventions::UEFN: return ConventionBasis3::LUF;
        case Conventions::Unity: return ConventionBasis3::RUF;
        case Conventions::USD: return ConventionBasis3::RUB;
        case Conventions::ZBrush: return ConventionBasis3::Unknown;
    }

    return ConventionBasis3::Unknown;
}
}
```