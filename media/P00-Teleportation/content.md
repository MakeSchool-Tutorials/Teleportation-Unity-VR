---
title: "Getting Started with Teleportation"
slug: wish-i-had-a-teleportal-gun
---

In this tutorial, we’ll implement a teleportation mechanic for a VR
puzzle game where the player solves puzzles, by teleporting onto
surfaces like walls and ceilings, to change which way gravity points.
The objective of each level is to move weighted Tokens to Goals.

![](../media/image89.gif)

VR is new and doesn’t have very many standards yet, but one that is
becoming popular is teleportation. We encourage you, for your own
projects, to try and think of a more creative alternative, but we do
think it’s useful to learn how to implement teleportation in case you
ever want to use it.

In this tutorial, we’ll implement:

-   A player that can launch out a beam and teleport onto surfaces!

-   Objects that change their direction of gravity

-   Non-teleportable and teleportable surfaces

-   A screen transition during teleportation

If you’d like to go on after the basic tutorial, we’ll also show you how
to make:

-   Beam-bouncing surfaces, like mirrors

-   Beam-bending objects, like prisms!

To begin, create a new 3D Project in Unity named “Teleportation,” and
import SteamVR from the Asset Store.

![](../media/image84.png)

For our game, we’ll want a pretty standard player (a head and two
hands), so the simplest place to start is with the SteamVR \[Camera
Rig\] Prefab.

Go ahead and create the Player.

![](../media/image68.png)

We’ve renamed the \[Camera Rig\] to “Player,” because that’s more
descriptive to our game. We’ve also turned Player into a Prefab, so that
we can use it in more than one level without having to copy-paste it
every time or recreate it.

![](../media/image37.png)

Now we want some sort of test environment that we’ll be able to use to
teleport around. We’ve made an inverted box out of differently colored
Planes so that we could clearly distinguish each surface and so that we
could try teleporting to walls at many different angles from each other.

![](../media/image58.png)

Before you test it, be sure to remove Main Camera! Remember, the main
camera you use is the first one Unity finds that’s tagged “MainCamera,”
so it’s best to avoid having multiple in the Scene if they’re all
active.

![](../media/image53.gif)

We’re going to teleport the player using a point-and-shoot mechanic in
which the Player will aim a hand at the ground and press a button on the
controller to teleport to that spot, if the player can. In order to both
test this and to give real players feedback, we’re going to project a
Laser and Reticle onto the surface at which we’re pointing and encode it
with visual information so that the player can look and see “Oh hey! I
can totally teleport there” or “Looks like a I can’t teleport there,
better try something else.”

SteamVR provides a built-in Laser Pointer (SteamVR/Extras/LaserPointer),
which you can drag onto your hand object directly, but we’re going to
build our own ;). If you’d like to see it though, feel free to play
around with the laser pointer, just be sure to remove it when you’re
done.

Our eventual goal is to make a system in which both hands can work as a
teleportation gun, but in order to get things up-and-running quickly,
we’re going to start with just one hand and then modify our design to
make it more general. This approach is a pretty good one to prevent
over-engineering a design; build just what you need, and then modify it
to fit your needs.

First, we’re going to make reticle that appears on the wall at the point
at which we’re aiming.

Create and add a TeleportationBeam component to whichever hand you’d
like to use for testing (we used our left) and put the following code
into it:

```
using UnityEngine;
using System.Collections;

public class TeleportationBeam : MonoBehaviour {

  public Transform reticle;

  // Use this for initialization
  void Start () {

  }

  // Update is called once per frame
  void Update () {

    RaycastHit hit;
    Ray ray = new Ray(transform.position, transform.forward);

    if (Physics.Raycast(ray,out hit)) {
      reticle.position = hit.point;
    }
  }
}
```

Then create and assign a Reticle to the component’s Reticle slot. We
used a Point Light with haloing turned on as your Reticle.

![](../media/image111.png)

![](../media/image42.gif)

Physics.Raycast is a method that returns true if a Ray (a geometric body
with both position and direction, so like a vector that never ends and
has a position in space) passes through a Collider, like those on our
Planes and Cubes. If a Collider is hit, information from that hit is
passed into the funny looking parameter “out hit” in the form of a
RaycastHit. The “out” means that this variable can be modified by the
function and then comes back to us as output. For a more technical
definition, “out” means the variable is passed by reference and will be
initialized within the function.

For our Ray, we’ve chosen one that starts at the hand’s position and
points the way our hand is pointing.

There’s an edge case our code misses though. If you change your level
such that there are gaps between the walls, you can see it. Watch what
happens when you drag your aim across a gap.

![](../media/image100.gif)

The reticle appears to “stick” to its old position when we pass over
gaps, i.e. when we’re not aiming at anything.

In order to fix this, we not only have to identify what to do
technically, but also have to make a design decision: what should the
reticle do when the player isn’t aiming at anything?

How do you want to resolve this issue? Go ahead and implement a fix.
Some helpful things to note: there are lots of forms of Physics.Raycast.
Our implementation makes use of adding a range parameter, but there are
lots of others.

We decided that our beam should have some maximum distance, and that if
the beam doesn’t hit a wall, it counts as a miss. To show a miss, we
decided to show the reticle at that max distance, but made it turn red.

![](../media/image97.gif)

We implemented this by changing our code and setting the public
variables in the Editor:

```
using UnityEngine;
using System.Collections;

public class TeleportationBeam : MonoBehaviour {

  public Transform reticle;
  public float range;

  public Color enabledColor;
  public Color disabledColor;

  private Light reticleLight;

  // Use this for initialization
  void Start () {
    reticleLight = reticle.gameObject.GetComponent<Light>();
  }

  // Update is called once per frame
  void Update () {

    RaycastHit hit;
    Ray ray = new Ray(transform.position, transform.forward);

    reticle.position = ray.origin + ray.direction * range;
    reticleLight.color = disabledColor;

    if (Physics.Raycast(ray, out hit, range)) {

      reticle.position = hit.point;

      reticleLight.color = enabledColor;
    }
  }
}
```

![](../media/image95.png)

Now it’s time to create a laser that extends from our hand to the
Reticle.

To do this, we can use the Line Renderer component. The Line Renderer
component renders a line through any set of points you specify for it.

If you want to see what it looks like, you can create an Empty Game
Object with a Line Renderer on it and try playing around with its
properties. You can change the line width at the beginning and end, tell
it to travel through a new set of positions, change how many positions
it travels through, etc. If you want to change the color, you’ll need to
give it a Material with a Shader that will let the colors show up. One
that works well is Particles/Alpha Blended Premultiply.

![](../media/image49.png)

Try adding code to make a laser beam that extends from your hand to the
Reticle. As a hint, given a Line Renderer lr and a generic List of
Vector3s, waypoints, you can set its positions by saying:

```
lr.SetVertexCount(waypoints.Count);
lr.SetPositions(waypoints.ToArray());
```

![](../media/image93.gif)

We changed our code to look like this:

```
using UnityEngine;
using System.Collections;

using System.Collections.Generic;

public class TeleportationBeam : MonoBehaviour {

  public Transform reticle;
  public LineRenderer laser;
  public float range;

  public Color enabledColor;
  public Color disabledColor;

  private Light reticleLight;

  // Use this for initialization
  void Start () {
    reticleLight = reticle.gameObject.GetComponent<Light>();
  }

  // Update is called once per frame
  void Update () {

    RaycastHit hit;
    Ray ray = new Ray(transform.position, transform.forward);

    List<Vector3> waypoints = new List<Vector3>();
    waypoints.Add(transform.position);

    reticle.position = ray.origin + ray.direction * range;

    reticleLight.color = disabledColor;
    laser.SetColors(disabledColor, disabledColor);

    if (Physics.Raycast(ray, out hit, range)) {

      reticle.position = hit.point;
      reticleLight.color = enabledColor;

      laser.SetColors(enabledColor, enabledColor);
    }

    waypoints.Add(reticle.position);

    laser.SetVertexCount(waypoints.Count);
    laser.SetPositions(waypoints.ToArray());
  }
}
```

The public Line Render variable was filled in the Editor by creating an
Empty Game Object with a Line Renderer (like the one we were playing
with!) and dragging it in.

Because we don’t want our laser to cast shadows, we turned off all
shadow-casting properties on the Line Renderer.

![](../media/image107.png)

Now we have some pretty good visual indicators to show where we’ll
teleport when we press the teleport button.

However, it’s kind of obnoxious to have to look at a laser shooting out
of our controller all the time, and it would be even more annoying with
two hands. We’d like our laser and reticle to only appear when we’re
holding down the teleport button and then for the actual teleportation
to occur when we release the button. Then, when we release it, we’d like
to actually teleport.

Since we don’t want to implement too many things at once, we’re first
going to take a baby step in this direction by making our beam only
appear when we hold down the teleport button (we’ll use the Trackpad),
and then doing… something with the hit information when we release that
button, for example, spawning a sphere at the hit point. Of course we’ll
take out this code when we implement the actual teleport mechanic, but,
for now, spawning a sphere is a very visual thing we can do to let
ourselves know we’ve done something correct with the data.

Go ahead and make the Laser and Reticle both only appear when you’re
holding down the Trackpad, and then make a sphere appear when you
release it.

Here are a few hints to get you started, since this can be a bit
involved (“Baby step” may apply better if it’s a baby hippo):

Given a SteamVR\_TrackedObject, controller, you can check whether or not
the Trackpad is currently pressed in the following way:

```
SteamVR_Controller.Device device = SteamVR_Controller.Input((int)controller.index);

if (device.GetPress(Valve.VR.EVRButtonId.k_EButton_Axis0) {
  Debug.Log(“We’re pressing the trackpad!”);
}
```

And to check a trackpad release, it’s the same, but “GetPressUp” instead
of “GetPress.”

You can make Game Objects inactive (i.e. invisible) or active (i.e.
visible) by calling SetActive(isActive) on them, where “isActive” is a
bool.

Be sure to test it out and have a good time making some giant worm
beasts.

![](../media/image31.gif)

We implemented this by changing our code to look like the following:

```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class TeleportationBeam : MonoBehaviour {

  public Valve.VR.EVRButtonId buttonId = Valve.VR.EVRButtonId.k_EButton_Axis0;

  public Transform reticle;
  public LineRenderer laser;
  public float range;

  public Color enabledColor;
  public Color disabledColor;

  private Light reticleLight;

  private SteamVR_TrackedObject controller;

  private RaycastHit target;
  private bool canTeleport;

  // Use this for initialization
  void Start () {

    reticleLight = reticle.gameObject.GetComponent<Light>();
    controller = GetComponent<SteamVR_TrackedObject>();
  }

  // Update is called once per frame
  void Update () {

    laser.gameObject.SetActive(false);
    reticle.gameObject.SetActive(false);

    SteamVR_Controller.Device device =
    SteamVR_Controller.Input((int)controller.index);

    if (device.GetPress(buttonId)) {

      canTeleport = false;

      laser.gameObject.SetActive(true);
      reticle.gameObject.SetActive(true);

      RaycastHit hit;
      Ray ray = new Ray(transform.position, transform.forward);

      List<Vector3> waypoints = new List<Vector3>();
      waypoints.Add(transform.position);

      reticle.position = ray.origin + ray.direction * range;

      reticleLight.color = disabledColor;
      laser.SetColors(disabledColor, disabledColor);

      if (Physics.Raycast(ray, out hit, range)) {

        reticle.position = hit.point;

        reticleLight.color = enabledColor;
        laser.SetColors(enabledColor, enabledColor);

        target = hit;
        canTeleport = true;
      }

      waypoints.Add(reticle.position);

      laser.SetVertexCount(waypoints.Count);
      laser.SetPositions(waypoints.ToArray());
    }

    if (device.GetPressUp(buttonId) && canTeleport) {

      GameObject go = GameObject.CreatePrimitive(PrimitiveType.Sphere);
      go.transform.position = target.point;
    }
  }
}
```

In order to not allow an up press occurring after an invalid beam cast
to count as a teleport, we added the canTeleport flag, which we set to
false on every trackpad press. We only set it to true if we hit
something, so that only a valid press will lead to a valid up press.

We also decided to make the buttonId a public variable (with a default
value) so that we could set that from the Editor if we ever changed our
minds about which button we wanted to use as the teleport button. In
general, this is a pretty good idea, and, if our component weren’t
already named “TeleportationBeam” we would have picked a name more
descriptive of teleportation, like “teleportButtonId.” Given the context
though, it seemed redundant to add the word “teleport” in there.

Now it’s time to actually teleport our Player!

In order to do this, we’ll just need to replace the Sphere spawning code
with some code to position our Player at the target point. That means
we’ll need access to our Player’s transform.

Go ahead and replace the sphere-spawning code with code that repositions
your Player -- don’t worry about rotation just yet.

By the way, if you have a fear of heights, we don’t recommend looking
down when you teleport, since you’ll still be rotated facing “up.”

![](../media/image92.png)

We did this by adding a public variable to reference our Player’s
transform:

```
public Transform player;
```

Then changed the sphere-spawning code to be this line:

```
player.position = target.point;
```

Then we just dragged a reference of our Player into the resulting slot
we created.

![](../media/image114.png)

In order to reorient our Player, we just need to make our Player’s up
direction face the same as the normal of the target point we hit.
Luckily, our hit gives us access to the normal. You can access the
normal of any RaycastHit, hit, by saying “hit.normal.”

Go ahead and make the Player teleport to the correct orientation.

We did this by changing our teleport code to look like this:

```
transform.position = target.point;
transform.up = target.normal;
```

Now we have our basic teleport mechanic in place for one hand. We’re
going to now make it work on BOTH hands.

This means that we’ll want a second Laser and Reticle, BUT we may want
to change how our Laser or Reticle look at some point in the future, so
copy-pasting them isn’t a great option.

Prefabs to the rescue!

Turn your Laser and Reticle into Prefabs, and make your component
generate them rather than reference pre-existing instances in the Scene.
When you do this, your Scene should run the same as before!

After we turned our Laser and Reticle into Prefabs, we added two public
member variables to TeleportationBeam:

```
public GameObject reticlePrefab;
public GameObject laserPrefab;
```

And we made the old variables that referenced our Laser and Reticle
private:

```
private Transform reticle;
private LineRenderer laser;
```

Then we changed our Start method to look like this:

```
void Start() {

  GameObject laserObj = (GameObject)Instantiate(laserPrefab);
  GameObject reticleObj = (GameObject)Instantiate(reticlePrefab);

  laserObj.transform.SetParent(player);
  reticleObj.transform.SetParent(player);

  reticle = reticleObj.transform;
  laser = laserObj.GetComponent<LineRenderer>();

  reticleLight = reticle.gameObject.GetComponent<Light>();
  controller = GetComponent<SteamVR_TrackedObject>();
}
```

We dragged references to our Laser and Reticle from our Project Panel
into the new slots we made in the Editor:

![](../media/image44.png)

Then we deleted the original Laser and Reticle from our Scene.

Making the old variables private wasn’t strictly necessary to our
implementation, but we did it for two reasons: (1) the variables no
longer need to be accessed publicly, and (2) by making the variables
private, they no longer appear in the Inspector, which saves us some
visual clutter.

We also didn’t need to set the newly created objects to be children of
the player, but we did this so that the Hierarchy panel wouldn’t be
cluttered when we ran the game, and we’d be able to collapse Player down
nicely.

You may be wondering: why didn’t we just drag copies of the Prefabs into
the Hierarchy directly and access those rather than creating them
programmatically? Well, this would have created a need for us to do this
for EVERY hand we wanted to outfit with the TeleportationBeam. You may
be thinking, “a Player will only ever have two hands! That’s no big
deal!” and that’s true, but if you want to make a new level, you need to
make a new Player for that level. We could get around this if the
Reticle and Laser Prefab were nested under the Player Prefab -- then we
could just drag out a Player Prefab, and it would already have the other
Prefabs attached to it; HOWEVER nested Prefabs in Unity do NOT retain
their Prefab links. This means we’d lose the utility of them being
Prefabs in the first place.

Now to dual wield TeleportationBeams, all you should need to do is drag
a new TeleportationBeam component onto your other hand.

Do that, and set up any additional properties you need to set up. As a
tip, if you want to give your component’s public properties default
values, you can set them inline as assignments BEFORE you add the
component to an object, and that version of the component will have
them:

```
public float range = 20f;

public Color enabledColor = Color.white;
public Color disabledColor = Color.red;
```

Be sure to test to make sure both work!

![](../media/image117.gif)

Feel free to take a few moments to customize your Beam and Reticle.

![](../media/image109.gif)

We added a SteamVR\_PlayArea as a child of our Reticle and made our
Reticle reorient based on the expected teleportation orientation.

In doing this, we also found it helpful to encapsulate our Reticle and
Laser as components, so that we didn’t have so much confusing clutter in
our TeleportationBeam that was so specific to how the beam was drawn.

Our new components looked like this:

```
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(LineRenderer))]
public class Laser : MonoBehaviour {

  LineRenderer lr;

  // Use this for initialization
  void Start () {
    lr = GetComponent<LineRenderer>();
  }

  // Update is called once per frame
  void Update () {

  }

  public void SetColor(Color color) {
    lr.SetColors(color, color);
  }

  public void SetWaypoints(Vector3[] waypoints) {

    lr.SetVertexCount(waypoints.Length);
    lr.SetPositions(waypoints);
  }
}
```

```
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(Light))]
public class Reticle : MonoBehaviour {

  private Light halo;
  public SteamVR_PlayArea playArea;

  // Use this for initialization
  void Start () {
    halo = GetComponent<Light>();
  }

  // Update is called once per frame
  void Update () {

  }

  public void SetColor(Color color) {

    halo.color = color;
    playArea.color = color;
  }

  public void ShowPlayArea(bool doShow) {
    playArea.enabled = doShow;
    playArea.gameObject.SetActive(doShow);
  }
}
```

And our TeleportationBeam component changed to look like this:

```
using UnityEngine;
using System.Collections;
using System.Collections.Generic;

public class TeleportationBeam : MonoBehaviour {

  public Valve.VR.EVRButtonId buttonId = Valve.VR.EVRButtonId.k_EButton_Axis0;

  public GameObject laserPrefab;
  public GameObject reticlePrefab;
  public Transform player;

  private Reticle reticle;
  private Laser laser;

  public float range = 20f;

  public Color enabledColor = Color.white;
  public Color disabledColor = Color.red;

  private SteamVR_TrackedObject controller;

  private RaycastHit target;
  private bool canTeleport;

  // Use this for initialization
  void Start() {

    GameObject laserObj = (GameObject)Instantiate(laserPrefab);
    GameObject reticleObj = (GameObject)Instantiate(reticlePrefab);

    laserObj.transform.SetParent(player);
    reticleObj.transform.SetParent(player);

    reticle = reticleObj.GetComponent<Reticle>();
    laser = laserObj.GetComponent<Laser>();

    controller = GetComponent<SteamVR_TrackedObject>();
  }

  // Update is called once per frame
  void Update() {

    laser.gameObject.SetActive(false);
    reticle.gameObject.SetActive(false);

    SteamVR_Controller.Device device = SteamVR_Controller.Input((int)controller.index);

    if (device.GetPress(buttonId)) {

      canTeleport = false;

      laser.gameObject.SetActive(true);
      reticle.gameObject.SetActive(true);

      RaycastHit hit;
      Ray ray = new Ray(transform.position, transform.forward);

      List<Vector3> waypoints = new List<Vector3>();
      waypoints.Add(transform.position);

      reticle.transform.position = ray.origin + ray.direction * range;

      if (Physics.Raycast(ray, out hit, range)) {

        target = hit;
        canTeleport = true;

        reticle.transform.position = target.point;
        reticle.transform.up = target.normal;

      }

      waypoints.Add(reticle.transform.position);

      laser.SetWaypoints(waypoints.ToArray());

      Color color = canTeleport ? enabledColor : disabledColor;
      laser.SetColor(color);
      reticle.SetColor(color);
      reticle.ShowPlayArea(canTeleport);
    }

    if (device.GetPressUp(buttonId) && canTeleport) {

      player.position = target.point;
      player.up = target.normal;
    }
  }
}
```

The new components were added to our Laser and Reticle respectively, and
our Reticle’s overall structure was changed to include a child with a
SteamVR\_PlayArea component attached to it.

![](../media/image101.png)

We’re now going to take a brief break from teleportation to add some
gameplay elements. In particular, we’re going to add some Tokens that
will change their gravity based on the Player’s orientation, and a Goal
that will signal a win when the Token reaches it.

Go ahead and create a Goal Prefab and a Token Prefab. To distinguish
them, make the Goal clear and the Token opaque.

![](../media/image113.png)

We’re going to make our Goal work not just by detecting whether or not
it’s touched the Token, but whether or not the Token is inside its
bounds, within some threshold.

In order to allow ourselves to do this kind of detection, change Goal’s
Collider to be a Trigger.

![](../media/image104.png)

Then make the goal destroy the token when the token falls inside it. To
test this, it may be easiest to place the Token directly above the Goal.

![](../media/image102.gif)

To do this, we tagged our Token with the new tag “Token,”

![](../media/image45.png)

and added the following component to our Goal:

```
using UnityEngine;
using System.Collections;

public class Goal : MonoBehaviour {

  public float threshold;

  void OnTriggerStay(Collider col) {

    if (col.CompareTag("Token")) {

      float distSq = (col.transform.position - transform.position).sqrMagnitude;

      if (distSq <= Mathf.Pow(threshold,2f)) {
        Destroy(col.gameObject);
      }
    }
  }

}
```

With threshold set to 0.5 in the Inspector.

The final two steps our gameplay needs is to make all the Tokens be
affected by a gravitational field relative to the Player, and to make
the game restart when there are no more tokens left!

Go ahead and make your Tokens change gravity based on the Player’s
direction. We suggest using the ConstantForce component -- a component
that allows you to apply a constant force in ANY direction -- and
turning off gravity on the Token’s Rigidbody.

![](../media/image91.gif)

To do this, we created a component a Game Object with a new component,
ScenePlay, attached to it that’s defined like this:

```
using UnityEngine;
using System.Collections;

public class ScenePlay : MonoBehaviour {

  public Transform player;
  public ConstantForce token;

  // Use this for initialization
  void Start () {

  }

  // Update is called once per frame
  void Update () {
    token.force = -player.up \* Physics.gravity.magnitude;
  }

}
```

The Token and Player were, of course, dragged into the slots that
appeared in the Inspector.

Now to make the game restart, we just need to check whether or not our
Token is still alive.

Go ahead and make the Scene restart when there are no more Tokens
remaining. Be sure you Build lights if you don’t want your Scene’s
lighting to change when it loads!

We implemented our win by changing ScenePlay to look like this:

```
using UnityEngine;
using System.Collections;
using UnityEngine.SceneManagement;

public class ScenePlay : MonoBehaviour {

  public Transform player;
  public ConstantForce token;

  private bool didEnd;

  // Use this for initialization
  void Start () {

  }

  // Update is called once per frame
  void Update() {

    if (!token && !didEnd) {

      didEnd = true;
      SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }

    if (didEnd) { return; }

    token.force = -player.up * Physics.gravity.magnitude;
  }
}
```

We know that our token’s been destroyed if it’s null.

We now have the core of our game, however, in implementing new features,
we’ve exposed some issues with our teleportation that are specific to
our gameplay.

First of all, our teleportation implementation allows us to teleport to
anything. Like, ANYTHING, including the Token and the Goal.

We don’t want our Player to be able to teleport to the goal, because…

![](../media/image12.gif)

AAAAAAHHH WHERE DID MY FLOOR JUST GO??? \*\*hurl\*\*

Further, because our Goal acts like a region rather than a solid object,
we really don’t want our beam to interact with it at all.

![](../media/image61.png)

First, let’s address our Goal.

To make our beam not interact with our Goal, we can simply add the
parameter “QueryTriggerInteraction.Ignore” to our Physics.Raycast
method:

```
Physics.Raycast(ray, out hit, range,1,QueryTriggerInteraction.Ignore)
```

Go ahead and do that.

Now your Goal and Beam should no longer collide!

![](../media/image60.gif)

Physics.Raycast has many forms, and some of them allow us to specify
that we want to ignore Colliders that have isTrigger set to true, like
the Goal. This new form required us to set a layer mask parameter (that
mysterious 1), a set of flags represented as a 32-bit number, where each
bit corresponds to a layer set to true or false. We could have also
prevented ray casting onto the Goal by putting it on a separate layer
from everything else; we chose this solution largely as a matter of
preference (fewer steps, and it has the added benefit that our beam now
also ignores collisions with any other trigger Colliders we add to our
Scene).

Now for the Token.

For our Token, we don’t want to ignore collisions, because, logically,
the beam shouldn’t pass through that object. Instead, we want our beam
to hit, but not allow our player to teleport. Additionally, we want to
reflect this visually so that we can indicate to our player “no, you
cannot teleport there.”

To do this, we’ll want to make the design decision of whether we want
players to ONLY be able to teleport to teleportable surfaces or to only
NOT be able to teleport to NON teleportable surfaces. Either choice
could be beneficial, depending on which we want to prioritize:
intentionality or discovery. Only allowing players to teleport where we
want them reduces the chances of players finding exploits or bugs in our
levels, since we’ve had to specify where players can wind up, but…
exploits might lead to fun discover (both before and after release), and
we’re less likely to find them if we box ourselves in. Lots of popular
games have gained niche followings because of exploits, others have
grown acclaimed for their tight design. Many good games have a mix of
both intentionality and discovery; as is common with these things, it’s
not a binary.

For the purposes of this tutorial, we’re going to take the more
intentional approach and specify teleportable surfaces, but you’re
welcome to resolve this problem whichever way you prefer.

Go ahead prevent teleportation to the Token.

![](../media/image63.gif)}

We did this by adding a “Teleportable” tag to all our walls, and
changing the line where we were setting our canTeleport flag in
TeleportationBeam to be this instead:

```
canTeleport = hit.collider.CompareTag("Teleportable");
```

Now our teleport interacts more appropriately with our game.

As one final step, we’re going to add an additive dissolve (i.e. a
dip-to-white) effect that will appear in front of our faces when we
teleport.

We’re going to do this via a Camera effect.

Import the Standard Assets Effects package (Assets-&gt;Import
Package-&gt;Effects) and then add a Screen Overlay component to your
Camera (eye) Game Object. Set its intensity to 0 and Blend Mode to
Additive.

![](../media/image41.png)

The Screen Overlay component overlays an image (or color) to the screen.
You can play around with the Intensity and Blend Mode to get a rough
idea of how it works.

We’re going to create the fade effect using an Animation. This is
definitely NOT the quickest way to implement this effect, but we’re
going to show you how, since this is a relatively simple effect and we
want to get show you the basic flow for working with Animations in
Unity.

Select Camera (eye) and then open the Animations Window
(Window-&gt;Animation)

![](../media/image116.png)

Click the “Create” button and you’ll be prompted to save this new
Animation somewhere. Save it to a new folder named “Animations” and name
it “AdditiveDissolve\_Appear.” When you do, your Animation Window should
change and you should see not only the Animation saved to your
Animations folder, but also the controller.

![](../media/image73.png)

If you look in the Inspector, you’ll also notice that the Camera (eye)
Game Object now has an additional Animator component attached to it.

![](../media/image74.png)

You may also notice that the Play buttons up top are now red and that
the Apply button in the Inspector is greyed out. The red indicates that
you are in Record Mode, and that changes you make to your component may
be recorded as keyframes. When you’re in Record Mode, you cannot Apply
Prefabs. You can exit (or re-enter) Record Mode at any time by clicking
the red circle icon in the Animations Window or by closing the window.

![](../media/image115.png)

Now we’re going to create the animation of the white layer fading in,
which will play when we start teleporting.

In the Animation Window, select Add Property, and then navigate to the
Intensity property of the Screen Overlay and click the little plus
symbol. The property should appear with two keyframes in the timeline.

![](../media/image108.png)

The default keyframes set are a bit far apart, and we want our animation
to be relatively quick, so as not to make players impatient, so click
and drag the farthest keyframe closer, to reduce the animation length.

![](../media/image55.gif)

Now select the last keyframe and change the value from 0 to 5.

![](../media/image40.png)

If you select the Game View, you sholud be able to see this change, and
can even play it back (and stop it) with the Play arrow in the Animation
Window or by scrubbing.

![](../media/image46.gif)

If you try to play the game, you’ll notice that this animation loops
forever and never stops!

To fix that, select the animation in the Project Panel, and then uncheck
Loop Time in the Inspector.

![](../media/image112.png)

Now do the same thing to create an AdditiveDissolve\_Disappear animation
that starts white and ends clear. Be sure to set it also to not loop!

![](../media/image105.png)

All right, we have these animations, so what? How do we call them?

We’re going to call our animations indirectly by triggering transitional
states, which we’ll set up in the Animator Window.

Close the Animation Window, and open the Animator Window
(Window-&gt;Animator). Select Camera (eye) and you should see our
animations represented as a flow chart.

![](../media/image39.png)

Each node represents a state of the animation, and each arrow represents
a transition from one state to the next. If you like technical terms,
this is the representation of your animation as a finite state machine!
Any State represents any state in the machine, and Entry is the start
point of your animation. By default, it points to the first animation
you created.

Right-click on AdditiveDissolve\_Appear and select “Make Transition” and
then drag the arrow to AdditiveDissolve\_Disappear.

![](../media/image38.gif)

If you select the arrow you just created, you’ll see in the Inspector
properties of this transition.

![](../media/image96.png)

Because we have no conditions set, AdditiveDissolve\_Appear will lead
directly into AdditiveDissolve\_Disappear whenever it completes.

Now create a transition from Any State to AdditiveDissolve\_Appear.

![](../media/image69.png)

We’re going to add a condition to this transition using something
totally unrelated to colliders, but also called a Trigger. In this
sense, a Trigger is a boolean that gets reset to false at the end of the
frame, so whenever we set it, Unity knows that it was just set this
frame and can use it to trigger a one-time thing.

Different parameter types are useful for different types of animations.
Bools are good for states like “Idle” vs “Falling.” Ints work well for
things that inherently have levels built in, like “Fatigue 1,” “Fatigue
2”, etc. Floats are great for actions that measure the same value, like
“Walking” vs “Running,” which may depend on magnitude of velocity.
Because of their one-shot nature, Triggers are very handy for animations
that get, well… triggered… by various actions. You can also mix and
match parameters on transitions.

In the upper left, where you have the option for Layers or Parameters,
select Parameters, if it’s not already selected.

![](../media/image36.png)

Then click the + symbol, select trigger, and fill in the field with the
string “Appear”

![](../media/image110.png)

![](../media/image99.png)

Then select the transition from Any State to AdditiveDissolve\_Appear
and click the plus below “Conditions” to add the Trigger as a condition.

![](../media/image48.png)

As one final step, right-click AdditiveDissolve\_Disappear and set it as
the default animation. This will make our scene start with a nice dip
from white.

![](../media/image103.png)

To call this additive dissolve effect, given an instance of the
Animator, animator, we can trigger the animation to begin by calling:

```
animator.SetTrigger(“Appear”);
```

Go ahead and call the animation when you teleport.

![](../media/image57.gif)

We did this by creating a public member variable, which we populated by
dragging the reference into the Inspector:

```
public Animator additiveDissolveAnim;
```

Then we called the following right before teleporting:

```
additiveDissolveAnim.SetTrigger("Appear");
```

You may have noticed that our animation has a bit of a problem. It
doesn’t mask the movement as it should. Ideally, the player should move
from point A to point B while the fade is at its peak, i.e. when the
white layer is fully blocking the screen.

One way we could do this is by implementing an Animation callback, but
so as not to make any of our functional code rely on any of our cosmetic
stuff, we’re going to use something called a Coroutine, which allows us
to wait for a certain amount of time before running some code.

To do this, we’re going to want to reference the length of our Animation
Clip of the white layer appearing.

Add a public AnimationClip variable to TeleportationBeam and drag in
AdditiveDissolve\_Appear from the Project Panel to fill it in.

```
public AnimationClip additiveDissolveClip;
```

Then add the following below your Update method:

```
private IEnumerator Teleport() {

  additiveDissolveAnim.SetTrigger("Appear");

  yield return new WaitForSeconds(additiveDissolveClip.length);

  player.position = target.point;

  player.up = target.normal;
}
```

And change your teleportation code in the Update method to look like
this:

```
if (device.GetPressUp(buttonId) && canTeleport) {
  StartCoroutine(Teleport());
}
```

StartCoroutine starts the Teleport coroutine, which you can think of as
a special function that can keep track of its progress over time to do
things at specified times to simulate waiting. The teleport coroutine
then proceeds with our teleportation as usual.

That funny "yield" syntax is used to temporarily halt execution of the Coroutine this cycle; it will pick up next cycle when the specified time has passed.

Try it out! Doesn’t that feel much smoother?

![](../media/image86.gif)

If you’d like more practice with coroutines and animation timing, go
ahead and try making the game end with a dip-to-white.

We did this by changing our ScenePlay method to look like this (and, of
course, dragging in the appropriate objects ;) )

```
using UnityEngine;
using System.Collections;
using UnityEngine.SceneManagement;

public class ScenePlay : MonoBehaviour {

  public Transform player;
  public ConstantForce token;

  public Animator additiveDissolveAnim;
  public AnimationClip additiveDissolveClip;

  private bool didEnd;

  // Use this for initialization
  void Start () {

  }

  // Update is called once per frame
  void Update() {

    if (!token && !didEnd) {

      didEnd = true;
      StartCoroutine(Restart());
    }

    if (didEnd) { return; }

    token.force = -player.up \* Physics.gravity.magnitude;
  }

  private IEnumerator Restart() {

    additiveDissolveAnim.SetTrigger("Appear");

    yield return new WaitForSeconds(additiveDissolveClip.length);

    SceneManager.LoadScene(SceneManager.GetActiveScene().name);
  }
}
```

Now we have a basic teleport mechanic in a fairly simple game.

Feel free to make some cosmetic tweaks to take a break from all that
functionality! We decided to do some Toon shading, add a Sky Dome we
found on the Asset Store, and make our beam have a moving texture,
amongst other things :)

![](../media/image89.gif)

![](../media/image72.gif)

That’s it for the basic tutorial!

But why stop here?

We’ve got this fun beam, so why not make it do other things beams do,
like reflect (like off a mirror) and refract (like off a prism)?

In order to make our beam bounce, we’ll no longer be able to rely on a
single Raycast alone. Instead, depending on the surface we hit, we may
want to bounce again… and again… and again!

In order to do this, we’re going to write a recursive function that does
everything our code currently does: get a list of waypoints, get a
reference to the final hit, and tell us whether or not we can teleport
to that hit or not.

Before we even think about adding new logic, let’s try to rewrite our
code as a recursive function that returns true if it can teleport, false
if not, and that sets the hit and waypoints by reference.

As a tip for writing recursive functions, a good pattern to follow is to
have a function with as few as possible parameters call a helper
function (which will probably have many more parameters). The helper
function will then call itself recursively until the task is complete
and return its value to the function its helping when it reaches its end
condition.

This might look something like this pseudocode:

```
Value Foo(aParam) {
  return FooHelper(aParamInit,bParamInit,cParamInit,soManyParamsInit);
}

Value FooHelper(aParam,bParam,cParam,soManyParams) {

  if (condition) {
    Return stuff;
  }

  return FooHelper(++aParam,++bParam,++cParam,++soManyParams);
}
```

In order to pass by reference, you can use the ref or out keywords (like
how we use “out” in Physics.Raycast). “out” and “ref” are actually quite
similar; the main difference is that parameters passed with ref are
required to be assigned BEFORE they are passed in, whereas parameters
passed with out are required to assigned WITHIN the function itself.

Another good tip is to think of what your end condition will be first,
i.e. how are you going to terminate your recursion? Whatever this is
will probably be a parameter in your helper function that changes each
turn.

Then start by putting your code into the helper function and figuring
out which of it should stay there vs what should move up a level by
determining how much of it is actually general to every step and what of
it is specific to the first step.

Go ahead and rewrite our code to use a recursive call to handle the
waypoints, final point, and whether or not we can teleport.

Be sure to test it out thoroughly and go over all the edge cases!

When we did this, the relevant functions of our code came out looking
like this:

```
// Update is called once per frame
void Update() {

  SteamVR_Controller.Device device = SteamVR_Controller.Input((int)controller.index);

  if (device.GetPress(buttonId)) {

    canTeleport = false;

    laser.gameObject.SetActive(true);
    reticle.gameObject.SetActive(true);

    List<Vector3> waypoints = new List<Vector3>();

    if (CanTeleport(ref waypoints, ref target)) {
      canTeleport = true;
      reticle.transform.up = target.normal;
    }

    if (waypoints.Count > 0) {
      reticle.transform.position = waypoints[waypoints.Count - 1];
    }

    laser.SetWaypoints(waypoints.ToArray());
    Color color = canTeleport ? enabledColor : disabledColor;
    laser.SetColor(color);
    reticle.SetColor(color);
    reticle.ShowPlayArea(canTeleport);
  } else {
    laser.gameObject.SetActive(false);
    reticle.gameObject.SetActive(false);
  }

  if (device.GetPressUp(buttonId) && canTeleport) {
    StartCoroutine(Teleport());
  }
}

private bool CanTeleport(ref List<Vector3>; waypoints, ref RaycastHit final) {

  Ray ray = new Ray(transform.position, transform.forward);
  waypoints.Add(transform.position);

  return CanTeleportHelper(ref waypoints, ref final, range, ray, false);
}

private bool CanTeleportHelper(ref List<Vector3> waypoints, ref RaycastHit final, float rangeRemaining, Ray ray, bool success) {

  if (rangeRemaining <= 0 || success) {
    return success;
  }

  RaycastHit hit;

  Vector3 waypoint = ray.origin + ray.direction * rangeRemaining;

  if (Physics.Raycast(ray, out hit, rangeRemaining, 1,QueryTriggerInteraction.Ignore)) {

    final = hit;
    waypoint = final.point;

    rangeRemaining -= (ray.origin - hit.point).magnitude;

    if (hit.collider.CompareTag("Teleportable")) {
      rangeRemaining = 0;
      success = true;
    } else {
      rangeRemaining = 0;
    }
  } else {
    rangeRemaining = 0;
  }

  waypoints.Add(waypoint);

  return CanTeleportHelper(ref waypoints, ref final, rangeRemaining, ray, success);
}
```

You may be looking at a few curious lines in CanTeleportHelper and
wondering… why? In particular these:

```
if (hit.collider.CompareTag("Teleportable")) {
  rangeRemaining = 0;
  success = true;
} else {
  rangeRemaining = 0;
}
```

Yes, that’s right: in all cases, we’re setting our rangeRemaining to be
0. Range remaining is our end condition, so this will certainly make
sure we terminate after one step only, but… why do we bother writing it
in this weird way?

The reason is that we plan to add more conditions here, and, if we leave
off the else, we get this funny behaviour whenever we hit a surface to
which we cannot teleport, like the token:

![](../media/image35.gif)

This is because, without further instruction, we’re just recasting the
same ray over and over until we run out of range. What we plan to do is
change our input ray based on the wall type we just hit, and, in the
case that we do hit a wall of no type, we do, in fact, want to just cut
off the range and call it an unsuccessful, but conclusive, hit.

Now that we have our code ready and our functionality where we left it,
we can extend our method to include reflection. All we need to do is
pass in a reflected ray when we hit a surface that reflects.

Unity has a built-in function for us that makes getting that ray
painless. We get the direction of the new ray by calling:

```
Vector3 direction = Vector3.Reflect(ray.direction, hit.normal);
```

With this in mind, implement reflection! In order to do this, you’ll of
course need to put a test objects into the Scene.

![](../media/image94.gif)

We modified our helper method to look like this:

```
private bool CanTeleportHelper(ref List<Vector3> waypoints, ref RaycastHit final, float rangeRemaining, Ray ray, bool success) {

  if (rangeRemaining <= 0 || success) {
    return success;
  }

  RaycastHit hit;

  Vector3 waypoint = ray.origin + ray.direction * rangeRemaining;

  if (Physics.Raycast(ray, out hit, rangeRemaining, 1,QueryTriggerInteraction.Ignore)) {

    final = hit;
    waypoint = final.point;

    rangeRemaining -= (ray.origin - hit.point).magnitude;

    if (hit.collider.CompareTag("Teleportable")) {

      rangeRemaining = 0;
      success = true;

    } else if (hit.collider.CompareTag("Reflect")) {

      ray.origin = hit.point;
      ray.direction = Vector3.Reflect(ray.direction, hit.normal);
    } else {

      rangeRemaining = 0;
    }
  } else {

    rangeRemaining = 0;
  }

  waypoints.Add(waypoint);

  return CanTeleportHelper(ref waypoints, ref final, rangeRemaining, ray, success);
}
```

Then we added a Cube to our Scene and gave it a new Reflect tag.

Now for refraction! The set-up for refraction is a little more
challenging because it involves shooting a ray through an object from
the inside and having it collide with the reverse face. If you cast a
ray from within a mesh, like a Cube, however, this won’t happen; the ray
will not detect a collision with the surface unless it’s approaching
against the normal.

In order to aid you, we’ve written a script to generate an inverted Mesh
Collider. This will allow us to collide from both sides of any Mesh
without having to do Mesh-specific set-up for each mesh we want to turn
into a refractor.

In fact, we’ve created a few scripts that you may find helpful. You can
access them by cloning them from this repo and opening the resulting
package in your project.

Obtain and import the package, and then create a new Primitive. Give it
a new “Refract” tag, and add the Refract component to it. To make it
easier to see what’s going on inside, give it a semi-transparent
Material. If the Primitive you made has a default Collider that is not a
Mesh Collider, remove it.

![](../media/image87.png)

The Refraction component adds an inverted Mesh Collider at runtime. It
also conveniently tags the object with the tag “Refract.”

You may have noticed the one public variable in Refract; index of
refraction. What is this?

The index of refraction of a substance is a number that determines how
light travels through it. Every substance has one, and when light passes
between two substances that have different indices of refraction, the
light changes direction, or bends!

This is why images look all woobly from behind a water glass!

When light passes from one medium to another, the new direction at which
it travels can be determined by the following equation:

directionNew = n1/n2 \* (norm X (-norm X directionOld)) - Sqrt(1 -
(n1/n2)\^2 \* (magnitude squared of (norm X directionOld)))

where n1 and n2 are the indices of refractions of the 1st and 2nd media
respectively, the norm is the normal of the surface hit, and the
directions old and new are the incoming and outgoing direcitons.

(much thanks to
[*SnarkEffects*](http://www.starkeffects.com/snells-law-vector.shtml)!)

This much will help you refract a ray into our Refract object. For now,
don’t worry about dealing with it once it’s inside; just get it to
refract.

Try implementing the first part of refraction; getting the ray inside.
As a hint, you’ll probably need to add a new parameter to your helper
method to handle the index of refraction.

The index of refraction of air is 1, so you can use that as your initial
value. The index of refraction of glass is around 1.5, so you can use
that as the value in the object in your Scene.

![](../media/image82.gif)

As a good check to make sure the math is correct, if you set both your
initial index of refraction to 1 and your object’s index of refraction
to 1, the beam should appear to pass straight through the refractive
object.

![](../media/image106.gif)

We added the following code below the reflection conditional:

```
else if (hit.collider.CompareTag("Refract")) {

  ray.origin = hit.point;

  Refract refract = hit.collider.GetComponent<Refract>();

  float indexOfRefractionNext = refract.indexOfRefraction;

  Vector3 s = ray.direction;
  Vector3 n = hit.normal;
  float m = indexOfRefraction / indexOfRefractionNext;
  Vector3 nXs = Vector3.Cross(n, s);

  ray.direction = m * Vector3.Cross(n, -nXs) - n * Mathf.Sqrt(Mathf.Abs(1 - Mathf.Pow(m, 2f) * Mathf.Pow(nXs.magnitude,2f)));

  indexOfRefraction = indexOfRefractionNext;
}
```

In order to make our beam refract out correctly, we need only reset the
index of refraction to 1 when we exit the object, but… how do we know
from which direction we’ve hit it?

Glad you asked!

There’s a very convenient property of closed polygons; if you cast a ray
out from any point in the center of a closed polygon, you’ll hit an odd
number of points that lie on that object’s surface! (corners count as
two points, you cheaters!)

All we need to do is cast out a ray and see how many times it’s hitting
this object. If an odd number of times, we’ll reset indexOfRefraction to
1. If not, we’ll set it to be the index of refraction of this refractive
material.

(In the case that we’re going directly from one refractive material to
another, we’ll actually end up bridging across a very very tiny gap of
index of refraction 1 using this method).

So how can we tell if the object we hit is the same one as any other
object? There’s a very definitive way to do this in Unity; object IDs.
Each Game Object has an object ID you can access by calling
go.GetInstanceID() for some instance of a GameObject, go.

Go ahead and implement this last bit to complete your refraction!

![](../media/image76.gif)

Our code looks like this:

```
ray.origin = hit.point + ray.direction \* wallThickness;

Refract refract =
hit.collider.gameObject.GetComponent<Refract>();

int numHits = 0;

RaycastHit[] hitChecks = Physics.RaycastAll(ray);
foreach (RaycastHit hitCheck in hitChecks) {
 if (hitCheck.collider.gameObject.GetInstanceID().Equals(hit.collider.gameObject.GetInstanceID())) {
   ++numHits;
 }
}

bool isInside = numHits % 2 == 0;

float indexOfRefractionNext = isInside ? 1f : refract.indexOfRefraction;

Vector3 s = ray.direction;
Vector3 n = hit.normal;
float m = indexOfRefraction / indexOfRefractionNext;
Vector3 nXs = Vector3.Cross(n,s);

ray.direction = m * Vector3.Cross(n,-nXs) - n * Mathf.Sqrt(Mathf.Abs(1 - Mathf.Pow(m,2f) * Mathf.Pow(nXs.magnitude,2f)));

indexOfRefraction = indexOfRefractionNext;
```

The variable wallThickness is a new public variable we added in order to
prevent a fringe bug that occurred in some cases due to float
imprecision. We set its value to be very very tiny: 1 x 10\^-6.

Congratulations! You’ve just implemented a beam that can reflect and
refract also known as... a Ray Tracer! If that word sounds familiar,
it’s because Ray Tracing is the process lighting uses to determine pixel
colors. ;)

If you’d like a light challenge, go ahead and try implementing a few
levels.

If you’d like a more difficult challenge, here are a few things to
consider:

Right now, our teleportation is a bit non-ideal. When we teleport onto a
wall, we land facing an arbitrary direction -- you can see this if you
have a Play Area on your reticle. This is because our method of
specifying orientation was imprecise: we specified an up direction, but
that still left us a degree of freedom, and so Unity snapped us to the
default orientation for that up direction. Can you think of a sensible
way to make the player land in a more expected orientation?

Right now, you can actually walk through the walls if you teleport next
to them and have enough play space to move around in the real world! It
would be ideal if the camera did something to let us know that we were
outside of the game’s intended bounds. Can you think of something to
resolve this issue?

And, of course, there’s always more cosmetic stuff to do ;p

Have fun!
