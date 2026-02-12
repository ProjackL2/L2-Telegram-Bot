```java
public interface IGameServerApi {
    String getVersion();
    long getUptime();
    long getTotalMemory();
    long getFreeMemory();
    double getCpuUsage();
    int getOnlinePlayers();
    int getMaxPlayers();

    int getCharacterId(String name);
    String getCharacterName(int characterId);
    ICharacterApi getCharacter(int characterId);
    long getDroppedAdena(int characterId);
    Map<String, Long> getDroppedItems(int characterId);
    void reportLinkSuccess(int characterId, String username);
}
```
