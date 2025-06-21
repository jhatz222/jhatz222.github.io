---
layout: page
title: "Databases"
permalink: /databases
---

## Artifact: Android Inventory Manager App

**Original:** Spring 2024 CS-360 in-memory inventory manager.  
**Enhancement:** Integrated persistent local storage with Room.

I replaced the volatile in-memory list of inventory items with a full Android Room database. I defined an `@Entity` class for items, a `@Dao` interface for queries, and an `@Database` class to wire them together. I wrapped all data access in a singleton `DataRepository`, then updated each activity to fetch from the repository rather than raw lists. Finally, I added the Room dependencies in Gradle so the app can compile and persist data across launches.

```java
// Item.java — Entity definition
@Entity(tableName = "items")
public class Item {
    @PrimaryKey(autoGenerate = true)
    public int id;
    public String name;
    public Date dueDate;
    // Constructors, getters, and setters omitted for brevity
}

// ItemDao.java — Data access object
@Dao
public interface ItemDao {
    @Query("SELECT * FROM items")
    List<Item> getAll();
    @Insert
    void insert(Item item);
    @Delete
    void delete(Item item);
}

// AppDatabase.java — Room database
@Database(entities = {Item.class}, version = 1)
public abstract class AppDatabase extends RoomDatabase {
    public abstract ItemDao itemDao();
}

// DataRepository.java — Singleton wrapper
public class DataRepository {
    private static volatile DataRepository instance;
    private final ItemDao dao;

    private DataRepository(Context ctx) {
        AppDatabase db = Room.databaseBuilder(
            ctx.getApplicationContext(),
            AppDatabase.class, "inventory-db"
        ).build();
        dao = db.itemDao();
    }

    public static DataRepository getInstance(Context ctx) {
        if (instance == null) {
            synchronized (DataRepository.class) {
                if (instance == null) {
                    instance = new DataRepository(ctx);
                }
            }
        }
        return instance;
    }

    public List<Item> getAllItems() {
        return dao.getAll();
    }
    // ...additional methods...
}
