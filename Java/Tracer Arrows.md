# Tracer Arrows

Old project that I revived to use on a server. Idea came from the [Machina](https://wiki.teamfortress.com/wiki/Machina) in Team Fortress 2, replicated by using particles.

## Table of Contents



## Particles

The main feature of this plugin is the tracer.  The tracer is generated in the [TAEvents](#TAEvents-Code) file, through a map of Arrow objects and co-ordinates.

When the arrow is shot, it is first checked if the shooter was a player. If not, it exits.

```java
@EventHandler
public void arrowShot(EntityShootBowEvent e) {
    if (!(e.getEntity() instanceof Player))
        return;
    Player p = (Player) e.getEntity();
    Arrow arrow = (Arrow) e.getProjectile();
    if (p.hasPermission("ta.use")) {
        if (Main.laser)
            arrow.setVelocity(arrow.getVelocity().multiply(Main.multiplier));
        arrow.setGravity(Main.gravity);
    }
    Location l = p.getLocation();
    Double[] temp = { Double.valueOf(l.getX()), Double.valueOf(l.getY()), Double.valueOf(l.getZ()) };
    this.arrows.put(arrow, temp);
}
```