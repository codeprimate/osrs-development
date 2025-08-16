# RuneLite Overlay Development Guide: Creating Custom UI Elements

## Table of Contents
1. [Understanding RuneLite's Overlay Architecture](#understanding-runelites-overlay-architecture)
2. [Types of Overlays](#types-of-overlays)
3. [Basic Overlay Implementation](#basic-overlay-implementation)
4. [Graphics2D Drawing Fundamentals](#graphics2d-drawing-fundamentals)
5. [Advanced Overlay Techniques](#advanced-overlay-techniques)
6. [Performance Optimization](#performance-optimization)
7. [Real-World Examples](#real-world-examples)
8. [Debugging and Testing](#debugging-and-testing)

## Understanding RuneLite's Overlay Architecture

### Core Concepts

Overlays in RuneLite extend the `Overlay` class and must override the `render()` method. They must be registered with the overlay manager on plugin startup and unregistered on shutdown. The overlay system operates as a layered rendering pipeline that draws on top of the game client.

### Overlay Lifecycle

```java
// Plugin startup
@Override
protected void startUp()
{
    overlayManager.add(myOverlay);
}

// Plugin shutdown  
@Override
protected void shutDown()
{
    overlayManager.remove(myOverlay);
}
```

### Key Architecture Components

1. **OverlayManager**: Handles overlay registration, positioning, and rendering order
2. **OverlayRenderer**: Core rendering engine that processes all overlays
3. **OverlayLayer**: Defines rendering depth (UNDER_WIDGETS, ABOVE_SCENE, etc.)
4. **OverlayPosition**: Controls overlay placement (DYNAMIC, DETACHED, TOP_LEFT, etc.)

## Types of Overlays

### 1. Panel Overlays (Information Boxes)

Text boxes are made via OverlayPanel, which provides structured information display. These are ideal for showing structured data like stats, timers, or configuration information.

```java
public class InfoOverlay extends OverlayPanel
{
    private final Client client;
    private final PluginConfig config;
    
    @Inject
    InfoOverlay(Client client, PluginConfig config)
    {
        this.client = client;
        this.config = config;
        setPosition(OverlayPosition.TOP_LEFT);
    }
    
    @Override
    public Dimension render(Graphics2D graphics)
    {
        panelComponent.getChildren().clear();
        
        // Add title
        panelComponent.getChildren().add(TitleComponent.builder()
            .text("My Plugin Info")
            .color(Color.GREEN)
            .build());
            
        // Add information lines
        panelComponent.getChildren().add(LineComponent.builder()
            .left("World:")
            .right(Integer.toString(client.getWorld()))
            .build());
            
        return panelComponent.render(graphics);
    }
}
```

### 2. Scene Overlays (World Drawing)

Overlays can draw anything they want anywhere on the game, such as clickboxes on game objects. Scene overlays draw directly onto the 3D game world.

```java
public class SceneOverlay extends Overlay
{
    private final Client client;
    
    @Inject
    SceneOverlay(Client client)
    {
        this.client = client;
        setPosition(OverlayPosition.DYNAMIC);
        setLayer(OverlayLayer.ABOVE_SCENE);
    }
    
    @Override
    public Dimension render(Graphics2D graphics)
    {
        // Draw on game objects, NPCs, players, etc.
        for (NPC npc : client.getNpcs())
        {
            if (npc.getName() != null && npc.getName().contains("Goblin"))
            {
                renderNpcOverlay(graphics, npc);
            }
        }
        return null;
    }
    
    private void renderNpcOverlay(Graphics2D graphics, NPC npc)
    {
        Polygon poly = npc.getCanvasTilePoly();
        if (poly != null)
        {
            OverlayUtil.renderPolygon(graphics, poly, Color.RED);
        }
        
        Point textLocation = npc.getCanvasTextLocation(graphics, "Target", 0);
        if (textLocation != null)
        {
            OverlayUtil.renderTextLocation(graphics, textLocation, "Target", Color.WHITE);
        }
    }
}
```

### 3. Widget Item Overlays

WidgetItemOverlay draws on items, allowing you to highlight or modify the appearance of inventory items, bank items, or any UI widget items.

```java
public class ItemOverlay extends WidgetItemOverlay
{
    @Inject
    ItemOverlay()
    {
        showOnInventory();
        showOnBank();
    }
    
    @Override
    public void renderItemOverlay(Graphics2D graphics, int itemId, WidgetItem widgetItem)
    {
        if (isHighPriorityItem(itemId))
        {
            Rectangle bounds = widgetItem.getCanvasBounds();
            graphics.setColor(new Color(255, 0, 0, 100));
            graphics.fill(bounds);
            graphics.setColor(Color.RED);
            graphics.draw(bounds);
        }
    }
    
    private boolean isHighPriorityItem(int itemId)
    {
        // Implement your item priority logic
        return itemId == ItemID.DRAGON_SCIMITAR || itemId == ItemID.RUNE_PLATEBODY;
    }
}
```

## Graphics2D Drawing Fundamentals

### Basic Drawing Operations

The `render()` method provides a `Graphics2D` object with full Java 2D drawing capabilities:

```java
@Override
public Dimension render(Graphics2D graphics)
{
    // Set rendering quality
    graphics.setRenderingHint(RenderingHints.KEY_ANTIALIASING, RenderingHints.VALUE_ANTIALIAS_ON);
    
    // Basic shapes
    graphics.setColor(Color.RED);
    graphics.fillRect(10, 10, 100, 50);
    graphics.setColor(Color.BLACK);
    graphics.drawRect(10, 10, 100, 50);
    
    // Text rendering
    graphics.setFont(new Font("Arial", Font.BOLD, 14));
    FontMetrics fm = graphics.getFontMetrics();
    String text = "Hello RuneLite!";
    int textWidth = fm.stringWidth(text);
    graphics.drawString(text, 10, 30);
    
    // Lines and polygons
    graphics.setStroke(new BasicStroke(2f));
    graphics.drawLine(0, 0, 100, 100);
    
    // Custom polygons
    Polygon triangle = new Polygon(new int[]{50, 75, 25}, new int[]{10, 50, 50}, 3);
    graphics.fillPolygon(triangle);
    
    return new Dimension(120, 60); // Return overlay size
}
```

### Color and Transparency

```java
// Solid colors
graphics.setColor(Color.RED);
graphics.setColor(new Color(255, 0, 0)); // RGB
graphics.setColor(new Color(255, 0, 0, 128)); // RGBA with transparency

// Gradient fills
GradientPaint gradient = new GradientPaint(
    0, 0, Color.BLUE,
    100, 100, Color.RED
);
graphics.setPaint(gradient);
graphics.fillRect(0, 0, 100, 100);

// Utility for consistent transparency
Color transparentRed = ColorUtil.colorWithAlpha(Color.RED, 128);
```

### Text Rendering with Shadows

RuneLite provides utility methods for text rendering with shadows:

```java
// Manual text with shadow
private void renderTextWithShadow(Graphics2D graphics, String text, int x, int y, Color color)
{
    // Draw shadow
    graphics.setColor(Color.BLACK);
    graphics.drawString(text, x + 1, y + 1);
    
    // Draw main text
    graphics.setColor(color);
    graphics.drawString(text, x, y);
}

// Using OverlayUtil (recommended)
Point textLocation = new Point(100, 100);
OverlayUtil.renderTextLocation(graphics, textLocation, "My Text", Color.WHITE);
```

## Advanced Overlay Techniques

### Dynamic Positioning and World Coordinate Conversion

Converting between world coordinates and screen coordinates is essential for drawing on game objects:

```java
// Convert world point to screen coordinates
WorldPoint worldPoint = new WorldPoint(3200, 3200, 0);
LocalPoint localPoint = LocalPoint.fromWorld(client, worldPoint);
if (localPoint != null)
{
    Point screenPoint = Perspective.localToCanvas(client, localPoint, 0);
    if (screenPoint != null)
    {
        graphics.setColor(Color.YELLOW);
        graphics.fillOval(screenPoint.getX() - 5, screenPoint.getY() - 5, 10, 10);
    }
}
```

### Minimap Drawing

OverlayUtil provides methods for drawing on the minimap:

```java
// Draw a dot on the minimap
WorldPoint targetLocation = new WorldPoint(3200, 3200, 0);
Point minimapPoint = Perspective.worldToMinimap(client, targetLocation);
if (minimapPoint != null)
{
    OverlayUtil.renderMinimapLocation(graphics, minimapPoint, Color.RED);
}
```

### Actor Overlays (NPCs and Players)

OverlayUtil provides convenience methods for rendering actor overlays:

```java
// Highlight an actor with polygon and text
for (Player player : client.getPlayers())
{
    if (player.getName().equals("TargetPlayer"))
    {
        OverlayUtil.renderActorOverlay(graphics, player, "TARGET", Color.RED);
    }
}

// Custom actor highlighting
private void highlightActor(Graphics2D graphics, Actor actor, Color color)
{
    Polygon poly = actor.getCanvasTilePoly();
    if (poly != null)
    {
        graphics.setColor(new Color(color.getRed(), color.getGreen(), color.getBlue(), 50));
        graphics.fill(poly);
        graphics.setColor(color);
        graphics.setStroke(new BasicStroke(2f));
        graphics.draw(poly);
    }
}
```

### Image Rendering

```java
// Load and draw images
private BufferedImage loadPluginImage(String imageName)
{
    try
    {
        return ImageIO.read(getClass().getResourceAsStream("/" + imageName));
    }
    catch (IOException e)
    {
        return null;
    }
}

// Render image at world location
BufferedImage icon = loadPluginImage("my_icon.png");
if (icon != null)
{
    LocalPoint localPoint = npc.getLocalLocation();
    OverlayUtil.renderImageLocation(client, graphics, localPoint, icon, 100);
}
```

### Custom Component Building

For complex panel overlays, build custom components:

```java
// Custom progress bar component
private LayoutableRenderableEntity createProgressBar(String label, int current, int maximum, Color color)
{
    return LineComponent.builder()
        .left(label)
        .right(current + "/" + maximum + " (" + (current * 100 / maximum) + "%)")
        .rightColor(color)
        .build();
}

// Usage in render method
panelComponent.getChildren().add(createProgressBar("Health", 75, 100, Color.GREEN));
```

## Performance Optimization

### Conditional Rendering

Only render when necessary to maintain performance:

```java
@Override
public Dimension render(Graphics2D graphics)
{
    // Check if we should render
    if (client.getGameState() != GameState.LOGGED_IN)
    {
        return null;
    }
    
    if (!config.showOverlay())
    {
        return null;
    }
    
    // Cache expensive calculations
    if (lastUpdateTick != client.getTickCount())
    {
        updateCachedData();
        lastUpdateTick = client.getTickCount();
    }
    
    // Render cached data
    renderCachedOverlay(graphics);
    return overlaySize;
}
```

### Efficient Collection Iteration

```java
// Avoid creating new lists every render
private final List<NPC> targetsCache = new ArrayList<>();

private void updateTargetCache()
{
    targetsCache.clear();
    for (NPC npc : client.getNpcs())
    {
        if (isValidTarget(npc))
        {
            targetsCache.add(npc);
        }
    }
}

@Override
public Dimension render(Graphics2D graphics)
{
    // Use cached list
    for (NPC npc : targetsCache)
    {
        renderNpcOverlay(graphics, npc);
    }
    return null;
}
```

### Graphics State Management

```java
// Save and restore graphics state for complex operations
private void renderComplexOverlay(Graphics2D graphics)
{
    // Save current state
    Stroke originalStroke = graphics.getStroke();
    Color originalColor = graphics.getColor();
    Composite originalComposite = graphics.getComposite();
    
    try
    {
        // Modify graphics settings
        graphics.setStroke(new BasicStroke(3f));
        graphics.setColor(Color.YELLOW);
        graphics.setComposite(AlphaComposite.getInstance(AlphaComposite.SRC_OVER, 0.7f));
        
        // Draw complex overlay
        drawComplexShape(graphics);
    }
    finally
    {
        // Restore original state
        graphics.setStroke(originalStroke);
        graphics.setColor(originalColor);
        graphics.setComposite(originalComposite);
    }
}
```

## Real-World Examples

### Health Bar Overlay

```java
public class HealthBarOverlay extends Overlay
{
    private final Client client;
    private final PluginConfig config;
    
    @Inject
    HealthBarOverlay(Client client, PluginConfig config)
    {
        this.client = client;
        this.config = config;
        setPosition(OverlayPosition.DYNAMIC);
        setLayer(OverlayLayer.ABOVE_SCENE);
    }
    
    @Override
    public Dimension render(Graphics2D graphics)
    {
        if (!config.showHealthBars())
        {
            return null;
        }
        
        for (NPC npc : client.getNpcs())
        {
            if (npc.getHealthRatio() != -1)
            {
                renderHealthBar(graphics, npc);
            }
        }
        return null;
    }
    
    private void renderHealthBar(Graphics2D graphics, NPC npc)
    {
        Point location = npc.getCanvasTextLocation(graphics, "", npc.getLogicalHeight() + 40);
        if (location == null)
        {
            return;
        }
        
        int healthRatio = npc.getHealthRatio();
        int healthScale = npc.getHealthScale();
        
        if (healthRatio < 0 || healthScale <= 0)
        {
            return;
        }
        
        double healthPercent = (double) healthRatio / healthScale;
        
        // Background
        graphics.setColor(Color.BLACK);
        graphics.fillRect(location.getX() - 25, location.getY() - 5, 50, 8);
        
        // Health bar
        Color healthColor = healthPercent > 0.6 ? Color.GREEN : 
                           healthPercent > 0.3 ? Color.YELLOW : Color.RED;
        graphics.setColor(healthColor);
        graphics.fillRect(location.getX() - 24, location.getY() - 4, (int)(48 * healthPercent), 6);
        
        // Border
        graphics.setColor(Color.WHITE);
        graphics.drawRect(location.getX() - 25, location.getY() - 5, 50, 8);
        
        // Health text
        String healthText = (int)(healthPercent * 100) + "%";
        FontMetrics fm = graphics.getFontMetrics();
        int textX = location.getX() - fm.stringWidth(healthText) / 2;
        OverlayUtil.renderTextLocation(graphics, new Point(textX, location.getY() + 2), healthText, Color.WHITE);
    }
}
```

### Minimap Waypoint System

```java
public class WaypointOverlay extends Overlay
{
    private final Client client;
    private final List<WorldPoint> waypoints;
    
    @Inject
    WaypointOverlay(Client client)
    {
        this.client = client;
        this.waypoints = new ArrayList<>();
        setPosition(OverlayPosition.DYNAMIC);
        setLayer(OverlayLayer.ABOVE_WIDGETS);
    }
    
    public void addWaypoint(WorldPoint point)
    {
        waypoints.add(point);
    }
    
    @Override
    public Dimension render(Graphics2D graphics)
    {
        LocalPoint playerLocation = client.getLocalPlayer().getLocalLocation();
        
        for (int i = 0; i < waypoints.size(); i++)
        {
            WorldPoint waypoint = waypoints.get(i);
            
            // Draw on minimap
            Point minimapPoint = Perspective.worldToMinimap(client, waypoint);
            if (minimapPoint != null)
            {
                drawWaypoint(graphics, minimapPoint, i + 1);
            }
            
            // Draw distance on main view if close
            LocalPoint waypointLocal = LocalPoint.fromWorld(client, waypoint);
            if (waypointLocal != null)
            {
                int distance = waypointLocal.distanceTo(playerLocation);
                if (distance < 1000) // Within reasonable distance
                {
                    Point screenPoint = Perspective.localToCanvas(client, waypointLocal, 0);
                    if (screenPoint != null)
                    {
                        drawWaypointMarker(graphics, screenPoint, i + 1, distance);
                    }
                }
            }
        }
        
        return null;
    }
    
    private void drawWaypoint(Graphics2D graphics, Point minimapPoint, int number)
    {
        // Outer circle
        graphics.setColor(Color.BLACK);
        graphics.fillOval(minimapPoint.getX() - 8, minimapPoint.getY() - 8, 16, 16);
        
        // Inner circle
        graphics.setColor(Color.CYAN);
        graphics.fillOval(minimapPoint.getX() - 6, minimapPoint.getY() - 6, 12, 12);
        
        // Number
        graphics.setColor(Color.BLACK);
        graphics.setFont(new Font("Arial", Font.BOLD, 10));
        FontMetrics fm = graphics.getFontMetrics();
        String text = String.valueOf(number);
        int textX = minimapPoint.getX() - fm.stringWidth(text) / 2;
        int textY = minimapPoint.getY() + fm.getAscent() / 2 - 1;
        graphics.drawString(text, textX, textY);
    }
    
    private void drawWaypointMarker(Graphics2D graphics, Point screenPoint, int number, int distance)
    {
        // Distance text
        String distanceText = distance / 128 + " tiles"; // Convert to tiles
        OverlayUtil.renderTextLocation(graphics, 
            new Point(screenPoint.getX(), screenPoint.getY() - 20), 
            "Waypoint " + number + " (" + distanceText + ")", 
            Color.CYAN);
            
        // Arrow pointing down to location
        graphics.setColor(Color.CYAN);
        graphics.setStroke(new BasicStroke(2f));
        graphics.drawLine(screenPoint.getX(), screenPoint.getY() - 10, screenPoint.getX(), screenPoint.getY());
        
        // Arrow tip
        graphics.fillPolygon(new int[]{screenPoint.getX() - 3, screenPoint.getX() + 3, screenPoint.getX()}, 
                           new int[]{screenPoint.getY() - 5, screenPoint.getY() - 5, screenPoint.getY()}, 3);
    }
}
```

## Debugging and Testing

### Debug Overlay Information

```java
public class DebugOverlay extends OverlayPanel
{
    private final Client client;
    
    @Inject
    DebugOverlay(Client client)
    {
        this.client = client;
        setPosition(OverlayPosition.TOP_RIGHT);
    }
    
    @Override
    public Dimension render(Graphics2D graphics)
    {
        if (!DEBUG_MODE) return null;
        
        panelComponent.getChildren().clear();
        
        panelComponent.getChildren().add(TitleComponent.builder()
            .text("Debug Info")
            .color(Color.YELLOW)
            .build());
            
        Player localPlayer = client.getLocalPlayer();
        if (localPlayer != null)
        {
            panelComponent.getChildren().add(LineComponent.builder()
                .left("Position:")
                .right(localPlayer.getWorldLocation().toString())
                .build());
                
            panelComponent.getChildren().add(LineComponent.builder()
                .left("Animation:")
                .right(String.valueOf(localPlayer.getAnimation()))
                .build());
        }
        
        panelComponent.getChildren().add(LineComponent.builder()
            .left("Game Tick:")
            .right(String.valueOf(client.getTickCount()))
            .build());
            
        return panelComponent.render(graphics);
    }
}
```

### Overlay Performance Monitoring

```java
// Add performance timing to your overlays
private long lastRenderTime = 0;
private long maxRenderTime = 0;
private int renderCount = 0;

@Override
public Dimension render(Graphics2D graphics)
{
    long startTime = System.nanoTime();
    
    try
    {
        // Your normal render code here
        return performActualRender(graphics);
    }
    finally
    {
        long renderTime = System.nanoTime() - startTime;
        lastRenderTime = renderTime;
        maxRenderTime = Math.max(maxRenderTime, renderTime);
        renderCount++;
        
        // Log performance every 1000 renders
        if (renderCount % 1000 == 0)
        {
            double avgMs = lastRenderTime / 1_000_000.0;
            double maxMs = maxRenderTime / 1_000_000.0;
            log.debug("Overlay performance - Last: {:.2f}ms, Max: {:.2f}ms", avgMs, maxMs);
        }
    }
}
```

## Common Patterns and Best Practices

### Configuration Integration

```java
// Always respect user configuration
@Override
public Dimension render(Graphics2D graphics)
{
    if (!config.isEnabled())
    {
        return null;
    }
    
    // Use configuration for styling
    Color overlayColor = config.getOverlayColor();
    boolean showText = config.showText();
    int opacity = config.getOpacity();
    
    // Apply configuration
    graphics.setColor(new Color(overlayColor.getRed(), overlayColor.getGreen(), 
                               overlayColor.getBlue(), opacity));
}
```

### Event-Driven Updates

```java
// Update overlay data based on game events
@Subscribe
public void onNpcSpawned(NpcSpawned event)
{
    if (isTargetNpc(event.getNpc()))
    {
        // Mark overlay for update
        overlayNeedsUpdate = true;
    }
}

@Subscribe
public void onNpcDespawned(NpcDespawned event)
{
    // Clean up overlay data
    removeNpcFromCache(event.getNpc());
}
```

### Memory Management

```java
// Properly clean up resources
@Override
protected void shutDown()
{
    overlayManager.remove(myOverlay);
    
    // Clear any cached data
    cachedOverlayData.clear();
    
    // Cancel any scheduled tasks
    if (updateTask != null)
    {
        updateTask.cancel();
    }
}
```

This comprehensive guide covers the practical aspects of creating RuneLite overlays, from basic concepts to advanced techniques. The key to successful overlay development is understanding the rendering pipeline, respecting performance constraints, and leveraging RuneLite's utility methods for common operations like coordinate conversion and drawing primitives.