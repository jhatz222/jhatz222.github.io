---
layout: page
title: Databases
permalink: /databases
---

# Databases

- **Original Artifact:** Spring 2024 CS-360 in-memory inventory manager  
- **Enhancement:** Integrated persistent local storage with Android Room  

---

## Purpose of App

The original inventory manager kept all items in a transient, in-memory list. On app restart everything was lost. With Room, each item is stored in a SQLite database on the device. Now:

- Items persist across launches  
- Queries can filter, sort, and index without reloading the entire list  
- The app is ready for more complex data operations (joins, migrations, etc.)

---

## Changes Made

- **Entity & DAO**  
  - Created `Item.java` annotated with `@Entity(tableName = "items")`  
  - Defined `ItemDao.java` interface annotated `@Dao` with `@Query`, `@Insert`, `@Delete` methods  
- **Database Setup**  
  - Built `AppDatabase.java` extending `RoomDatabase`, listing `entities = {Item.class}`  
  - Configured versioning (e.g. `version = 1`) for future migrations  
- **Repository Pattern**  
  - Wrapped all data access in `DataRepository.java` singleton  
  - Exposed CRUD methods (`getAllItems()`, `insert()`, `delete()`, etc.)  
- **UI Integration**  
  - Updated each Activity to fetch and observe data via `DataRepository`  
  - Handled Room initialization and permissions in a cleaner, centralized way  
- **Build Configuration**  
  - Added Room and lifecycle dependencies to `build.gradle.kts`

---

## Important Code Snippets

```java
// Item.java — Room entity
@Entity(tableName = "items")
public class Item {
  @PrimaryKey(autoGenerate = true)
  public int id;

  public String name;
  public Date dueDate;

  // (constructors, getters/setters omitted for brevity)
}

// ItemDao.java — data access object
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
@Database(entities = { Item.class }, version = 1)
public abstract class AppDatabase extends RoomDatabase {
  public abstract ItemDao itemDao();
}

// DataRepository.java — singleton wrapper
public class DataRepository {
  private static volatile DataRepository instance;
  private final ItemDao dao;

  private DataRepository(Context ctx) {
    AppDatabase db = Room.databaseBuilder(
      ctx.getApplicationContext(),
      AppDatabase.class,
      "inventory-db"
    ).build();
    dao = db.itemDao();
  }

  public static DataRepository getInstance(Context ctx) {
    if (instance == null) {
      synchronized (DataRepository.class) {
        if (instance == null)
          instance = new DataRepository(ctx);
      }
    }
    return instance;
  }

  public List<Item> getAllItems() {
    return dao.getAll();
  }

  public void insertItem(Item item) {
    dao.insert(item);
  }

  public void deleteItem(Item item) {
    dao.delete(item);
  }
}

// build.gradle.kts — Room and lifecycle deps
implementation("androidx.room:room-runtime:2.5.1")
kapt("androidx.room:room-compiler:2.5.1")
implementation("androidx.room:room-ktx:2.5.1")
implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.6.1")

---

## Why These Changes Were Made

- Persistence: Users expect their inventory to survive app restarts.

- Scalability: Room’s query power (indexes, filters) avoids manual list scans.

- Clean Architecture: Repository pattern centralizes data logic, making UI code lean and testable.

- Future-Proofing: Versioned migrations and structured DAOs prepare the app for offline sync or larger datasets.

---

##Reflection on Course Outcomes

- Data Management (Outcome 4): Leveraged Room to model and persist entities.

- Software Design (Outcome 1): Applied repository and DAO patterns for clear separation of concerns.

- Performance Awareness (Outcome 3): Indexed fields to optimize query execution and avoid full scans.

---