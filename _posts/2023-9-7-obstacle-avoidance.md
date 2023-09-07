### Collision Detection and Avoidance

Work continues on the asteroid sector, now that I'm pretty happy with how the debris sector plays out. The generation here is going to look fairly similar but as discussed in prior posts, the main difference is that the asteroids *move* and random ship-to-ship debris doesn't. Moving through it in the manner [I described last time](https://wizard-of-chaos.github.io/2023/07/26/conceptualization.html) means that you're much, much more likely to smack into a rock now and get pasted like Randy Johnson's bird.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/randy.gif "One of these days, I'll get sick of this gif. Probably not this year.")

At least it looks pretty now, though, between fog updates, skybox updates, some new debris and some new types of road-block in your path...

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/new_asteroid_look.png "Whoa! Are those space stations?")

I'm not entirely happy with the look of the dust clouds right now (too bright, too shiny, too unaffected by lighting) and have been working on a couple concepts to make them look better. Right now those are too bright, and my other working concept doesn't have a hell of a lot of color variance, so it looks a bit weird.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/dust_cloud_wip.png "Not great, not terrible")

I'll figure it out -- probably by either writing a cloud shader or mixing these two concepts. In any case, the asteroids and constant movement means it's *much* more likely to hit a rock, and much more likely for the AI to hit a rock. Unfortunately my obstacle avoidance code was pretty bad, written around last April and not updated or tweaked since then -- it was able to detect basic collisions, but ships had this frustrating habit of stopping just short of the obstacle and simply *staying* there, which on a *moving* object meant they just got crushed like beer cans. After losing wingman after wingman to avoidable collisions I realized I needed to just rewrite the damn thing. 

Proper collision detection and avoidance requires a few concepts, though. Thankfully [Bullet3](https://pybullet.org/wordpress/), the physics library I'm using, already wrote most of them for me, and I needed to figure out how to apply the various tools it has into a spaceship game.

## Detection

First and foremost, you need to figure out if you're going to hit something. If you're parked in the center of the road, you're probably not going to hit anything. Something might hit *you*, but you yourself aren't going to hit anything. In this case, figuring out if I'm about to hit something is very easy -- I'm not.

But if I'm *driving* down the road, collisions become much more likely.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/car_thump.gif "This video just sort of baffles me whenever I see it.")

In this case I have somewhere I'm trying to get and I have my own forward velocity to work off of. In either case, I have a *path* for myself -- the straight line between my current position, and either the place I'm trying to get or where I'm going to be in, say, five seconds based on my current velocity. Now I just need to find out if there's something *on* that path -- but how can I check that?

One possible method, to go back to our car metaphor, is to get an inflatable beach ball and hurl it out in front of us as we're driving. If the ball bounces off of something, we're about to collide with whatever it bounced off of, and we should brake or swerve. I wouldn't recommend doing this in an actual car, but for a computer simulation this is pretty much how it gets done. This is such a common use-case for physics that Bullet3 [has this feature already built in.](https://pybullet.org/Bullet/BulletFull/classbtClosestNotMeConvexResultCallback.html)

```cpp
const btCollisionObject* obstacleOnPath(btRigidBody* rb, const btVector3& target)
{
	if (!bWorld || !rb) return nullptr;

	const btVector3& pos = rb->getCenterOfMassPosition();
	btClosestNotMeConvexResultCallback cb = btClosestNotMeConvexResultCallback(rb, pos, target, bWorld->getPairCache(), bWorld->getDispatcher());

	btSphereShape shape = btSphereShape(rb->getCollisionShape()->getLocalScaling().x() + 5.f);

	btTransform from(rb->getOrientation(), pos);
	btTransform to(rb->getOrientation(), target);
	to.setOrigin(target);
	bWorld->convexSweepTest(&shape, from, to, cb);

	if (cb.hasHit()) return cb.m_hitCollisionObject;
	return nullptr;
}
```

In this case it's the btClosestNotMeConvexResultCallback, which is exactly what it says on the tin -- it throws a convex shape down the line I give it, and assuming it hits something that isn't "me" (in this case, the starting object) it'll return whatever it hit so I can look at it further.

We're going to be running these functions several times a second, so we don't want to use the actual collision object we have for the rigid body -- we want to save that for *actual* questions. In this case, all we need is a rough approximation of what's in front of us, and we can get that with a sphere. The way to determine if something hits a sphere is incredibly easy -- just check if you're within the radius of the sphere -- so we use that for speed reasons.

We throw our virtual sphere down the line we've specified with "from" and "to" and see if there's something in the way. If there is, congrats! You're about to become the proud owner of a car accident. The question then becomes, how do we avoid the object?

## Avoidance

We now know that we're about to hit something, and the question becomes what do we do to get out of the way. This actually gave me some mental trouble trying to define it. You need some way to define the *edge* of the object in question, and to do it quickly without a lot of checks. In this case, I'm going to want an [axis-aligned bounding box](https://en.wikipedia.org/wiki/Bounding_volume). 

If you're not aware of what one of these things is, let me put it this way: I'm six feet tall, a bit under two feet wide and about a half foot thick. My personal bounding box if I'm standing upright is a rectangular prism that has those dimensions. If I reach out my arms in front of me, however, my bounding box changes: I'm now about six feet tall, a bit under two feet wide, and about three or four feet thick. The box then changes for those dimensions. The wikipedia article has a great picture if you're having trouble visualizing this.

This is a very simple method to determine just where the edges of the object even are in a very rough sense, and is typically used for broad-phase collision checks (where you determine if objects are even close to one another before getting down to more serious collision checks). Once again, Bullet3 comes with a method built-in to get me this thing.

![alt text](https://raw.githubusercontent.com/Wizard-Of-Chaos/Wizard-of-Chaos.github.io/main/imgs/getaabb.png "Thank god I don't have to write this crap myself.")

This gives me the minimum point for the box and the maximum point for the box. As the person trying to avoid the obstacle, I want to swerve toward whichever is closer to get around it, and then give myself some padding room for good measure based on my own size.

```cpp
const btVector3 pathAroundObstacle(const btRigidBody* body, const btCollisionObject* obstacle, const btVector3& target)
{
	if (!body || !obstacle) return target; //wtf?

	btVector3 boxMin, boxMax, bodyPos, obstaclePos;
	bodyPos = body->getCenterOfMassPosition();
	obstaclePos = obstacle->getWorldTransform().getOrigin();

	obstacle->getCollisionShape()->getAabb(obstacle->getWorldTransform(), boxMin, boxMax);

	btVector3& closer = boxMin;
	//we don't care about details, we just want to know which is closer
	if (bodyPos.distance2(boxMin) > bodyPos.distance2(boxMax)) closer = boxMax;

	btScalar multVal = body->getCollisionShape()->getLocalScaling().x() * 5.f;
	btVector3 directionAway = (obstaclePos - closer).normalized();
	btVector3 pathPoint = closer + (directionAway * (multVal + 15.f));

	return pathPoint;
}
```

Our new target position becomes something far away from the bounding box, and we maintain this until we're no longer about to hit the object, at which point we resume the original course. This worked on the first try on initial implementation, and I was very pleased with the results -- situations that would previously have resulted in a squishing now are neatly avoided by my wingmen. Thank god. I was getting sick of people dying to rocks.

## Is There Room To Improve?

Yes, absolutely. For one thing, this only detects if *you are about to hit something*, not if something is about to hit *you*. In order to determine if something's about to hit *you*, you'd need to keep track of nearby obstacles and their velocity and then toss virtual beach balls down every single vector for all of them. This is a lot more computationally expensive (and gets far, far worse the more loosely you define "nearby"), so I'm not even going to bother with this. If your wingman is parked, and a rock is flying towards them, sucks to be them, I guess.

The other thing is that it only takes *two* points on the bounding box, when really to determine what the best swerve path is it should probably take all eight points on the rectangle, find the closest one and swerve towards that instead. This is a little more expensive but would result in better dodges, so I may implement that later on.

For better accuracy (and to allow for daring near-miss scenarios) you could use the actual collision shape for the rigid body instead of spheres to determine *juuuust* how well you could squeeze by someone, but again, this is far too expensive to run on the fly given the amount of objects and ships, so we won't. This could be more useful on a smaller scale.


## What's Next?

Finish up the look for the asteroid sector and continue onward into the reworking for the rest of the sectors as well as writing dialogue. Look for Extermination Shock on Steam or itch.io soon -- we're getting there!