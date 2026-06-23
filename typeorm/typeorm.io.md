
- [1. Getting Started](#1-getting-started)
- [2. Data Source](#2-data-source)
- [3. Entity](#3-entity)
- [4. Relations](#4-relations)
- [5. Migrations](#5-migrations)
- [6. Working with Entity Manager](#6-working-with-entity-manager)
- [7. Query Builder](#7-query-builder)
  - [7.1. Select using Query Builder](#71-select-using-query-builder)
- [8. Query Runner](#8-query-runner)
- [9. Guides](#9-guides)
- [10. Advanced Topics](#10-advanced-topics)


# 1. Getting Started

```ts
"emitDecoratorMetadata": true,
"experimentalDecorators": true,
```
```ts
@Entity()
export class Photo {
    @PrimaryGeneratedColumn()
    id: number
    // By default, the string is mapped to a varchar(255)-like type
    @Column({ length: 100 })
    name: string
    @Column("text")
    description: string
    @Column()
    filename: string
    @Column("double")
    views: number
    @Column()
    isPublished: boolean
}

import "reflect-metadata"

const AppDataSource = new DataSource({
    type: "postgres",
    host: "localhost",
    port: 5432,
    username: "root",
    password: "admin",
    database: "test",
    entities: [Photo],
    synchronize: true,
    logging: false,
})

try {
    await AppDataSource.initialize()
} catch (error) {
    console.log(error)
}

const photo = new Photo()
...
await AppDataSource.manager.save(photo) // using EntityManager to save
const savedPhotos = await AppDataSource.manager.find(Photo)

// using repository
const photoRepository = AppDataSource.getRepository(Photo)
const photoToUpdate = await photoRepository.findOneBy({id: 1})
photoToUpdate.name = "Me, my friends and polar bears"
await photoRepository.save(photoToUpdate)
```
```ts
// Creating a one-to-one relation

@Entity()
export class PhotoMetadata {
    @PrimaryGeneratedColumn()
    id: number
    @Column("int")
    height: number
    ...
    @OneToOne(() => Photo)
    @JoinColumn()
    photo: Photo
}
// +-------------+--------------+----------------------------+
// |                     photo_metadata                      |
// +-------------+--------------+----------------------------+
// | id          | int          | PRIMARY KEY AUTO_INCREMENT |
// | height      | int          |                            |
// | ...         | ...          | ...                        |
// | photoId     | int          | FOREIGN KEY                |
// +-------------+--------------+----------------------------+
```
`@JoinColumn` decorator, indicates that this side of the relationship will own the relationship (it will be foreign key). Relations can be unidirectional or bidirectional. Only one side of the relation can be the owner (foreign key).
```ts
const photo = new Photo()
...

const metadata = new PhotoMetadata()
...
metadata.photo = photo // this way we connect them

const photoRepository = AppDataSource.getRepository(Photo)
const metadataRepository = AppDataSource.getRepository(PhotoMetadata)

await photoRepository.save(photo)
await metadataRepository.save(metadata)
```
Currently, our relation between PhotoMetadata and Photo is unidirectional. The owner of the relation is PhotoMetadata, and Photo doesn't know anything about PhotoMetadata. This makes it complicated to access PhotoMetadata from the Photo side. To fix this issue, we should add an inverse relation, and make relations between PhotoMetadata and Photo bidirectional
```ts
@Entity()
export class PhotoMetadata {
    /* ... other columns */
    @OneToOne(() => Photo, (photo) => photo.metadata)
    @JoinColumn()
    photo: Photo
}

@Entity()
export class Photo {
    /* ... other columns */
    @OneToOne(() => PhotoMetadata, (photoMetadata) => photoMetadata.photo)
    metadata: PhotoMetadata
}
```
If you use ESM in your TypeScript project, you should use the Relation wrapper type in relation properties to avoid circular dependency issues.
```ts
    ...
    photo: Relation<Photo>

    ...
    metadata: Relation<PhotoMetadata>
```
```ts
// Loading objects with their relations
const photoRepository = AppDataSource.getRepository(Photo)
const photos = await photoRepository.find({
    relations: {
        metadata: true,
    },
})
```
We can set up cascade options in our relations, in the cases when we want our related object to be saved whenever the other object is saved.
```ts
export class Photo {
    // ... other columns
    @OneToOne(() => PhotoMetadata, (metadata) => metadata.photo, {cascade: true})
    metadata: PhotoMetadata
}

const photo = new Photo()
...
const metadata = new PhotoMetadata()
...
photo.metadata = metadata // this way we connect them

const photoRepository = AppDataSource.getRepository(Photo)
await photoRepository.save(photo)
```
Let's create a many-to-one/one-to-many relation. Let's say a photo has one author, and each author can have many photos.
```ts
@Entity()
export class Author {
    @PrimaryGeneratedColumn()
    id: number
    @Column()
    name: string
    @ManyToMany(() => Photo, (photo) => photo.albums)
    @JoinTable()
    photos: Photo[]
}
// +-------------+--------------+----------------------------+
// |                          author                         |
// +-------------+--------------+----------------------------+
// | id          | int          | PRIMARY KEY AUTO_INCREMENT |
// | name        | varchar(255) |                            |
// +-------------+--------------+----------------------------+

@Entity()
export class Photo {
    /* ... other columns */
    @ManyToOne(() => Author, (author) => author.photos)
    author: Author
}
// +-------------+--------------+----------------------------+
// |                         photo                           |
// +-------------+--------------+----------------------------+
// | id          | int          | PRIMARY KEY AUTO_INCREMENT |
// | ...         | ...          | ...                        |
// | authorId    | int          | FOREIGN KEY                |
// +-------------+--------------+----------------------------+
```
`Author` contains an inverse side of a relation. `OneToMany` is always an inverse side of the relation, and it can't exist without `ManyToOne` on the other side of the relation. the owner side is always many-to-one.

Let's create a many-to-many relation. Let's say a photo can be in many albums, and each album can contain many photos.
```ts
@Entity()
export class Album {
    @PrimaryGeneratedColumn()
    id: number
    @Column()
    name: string
    @ManyToMany(() => Photo, (photo) => photo.albums)
    @JoinTable()
    photos: Photo[]
}
export class Photo {
    // ... other columns
    @ManyToMany(() => Album, (album) => album.photos)
    albums: Album[]
}
```
After you run the application, the ORM will create an **album_photos_photo_albums** junction table:
```sql
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int          | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int          | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```
```ts
const album1 = new Album()
await AppDataSource.manager.save(album1)
const album2 = new Album()
await AppDataSource.manager.save(album2)

const photo = new Photo()
photo.albums = [album1, album2]
await AppDataSource.manager.save(photo)

const loadedPhoto = await AppDataSource.getRepository(Photo).findOne({
    where: { id: 1},
    relations: { albums: true},
})
```
You can use QueryBuilder to build SQL queries of almost any complexity
```ts
const photos = await AppDataSource.getRepository(Photo)
    // initializes query against the Photo table and assigns it an alias
    .createQueryBuilder("photo")
    // This joins the one-to-one relation and populates an photo.metadata property
    .innerJoinAndSelect("photo.metadata", "metadata")
    // This joins the many-to-many relation and populates an photo.albums array
    .leftJoinAndSelect("photo.albums", "album")
    .where("photo.isPublished = true")
    .andWhere("(photo.name = :photoName OR photo.name = :bearName)")
    .orderBy("photo.id", "DESC")
    .setParameters({ photoName: "My", bearName: "Mishka" })

// SELECT * FROM photos photo
// INNER JOIN photo_metadata metadata ON metadata.photoId = photo.id
// LEFT JOIN photo_albums_albums junction ON junction.photoId = photo.id
// LEFT JOIN albums album ON junction.albumId = album.id
// WHERE photo.isPublished = true;
//  AND (photo.name = 'My' OR photo.name = 'Mishka')
// ORDER BY photo.id DESC;
```

# 2. Data Source
...

# 3. Entity
...

# 4. Relations
...

# 5. Migrations
...

# 6. Working with Entity Manager
...

# 7. Query Builder

## 7.1. [Select using Query Builder](https://typeorm.io/docs/query-builder/select-query-builder)

- There are two types of results you can get using select query builder: entities or raw results.

- `createQueryBuilder("user")` is equivalent to: `createQueryBuilder().select("user").from(User, "user")`
  
  `SELECT ... FROM users user`
...

...

# 8. Query Runner

# 9. Guides

# 10. Advanced Topics
