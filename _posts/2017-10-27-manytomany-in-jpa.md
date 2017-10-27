---
layout: post
title:  "Implement a many-to-many relationship in JPA (Hibernate)"
date:   2017-10-27 10:40:17 +0200
categories: java
---
Modern applications do not use separate scripts to create and manage their relational database: 
they use instead an [Object-to-Relational-Mapper] (ORM). Java Enterprise defines a protocol for ORMs
that is called Java Persistance API ([JPA]). One of the most famous [JPA] implementations is [Hibernate].

Implementing basic [one-to-one] or [one-to-many] relationships using [JPA] is fairly simple.

More tricky is to implement a *many-to-many* relationship. A many-to-many is described by a direct relationship (the
relation between the first entity with the second), and an inverse relatioship (the relation between the second
entity with the first one).

Basically, many-to-many relationship is implemented in SQL by creating a "junction table", that is a table
the joins the ID of the first entity with the ID of the second entity.

![Many-to-Many Relationship]({{ site.url }}/assets/manytomany.jpg)

Example of many-to-many are the following:

1. A Book is written by some Authors (The Author has written many Books)
2. A User Role has many Permissions (The Permission is given to many Roles)

How to implement such a relationship in Java using JPA?

Let's talk about the relation between Role and Permissions. Normally an application has users divided into groups,
these groups have a "role". Each "role" is given one or more "permissions", ie: read, write, update, delete.

In our example the first entity is the Role. We can define it as follows, using [Kotlin] language:

{% highlight kotlin %}
@Entity
open class Role : Serializable
{
    @Id
    var id: Long = 0L
        private set

    @Column(unique = true)
    var description: String = ""

    @OneToMany(mappedBy = "role", cascade = arrayOf(CascadeType.ALL))
    var users: MutableSet<User> = mutableSetOf()

    @ManyToMany
    var permissions: MutableSet<Permission> = mutableSetOf()
    
}
{% endhighlight %}

We have defined the entity role, which has a primary key called "id", a column with unique values called
"description", a one-to-many relation with the entity User (a Role is associated with many users) and
finally we have our many-to-many relationship with "Permission", that can as well be defined in the following way:

{% highlight kotlin %}
@Entity
open class Permission : Serializable
{
    @Id
    var id: Long = 0L
        private set

    @Column(unique = true)
    var description: String = ""

    @ManyToMany(mappedBy = "permissions")
    var roles: MutableSet<Role> = mutableSetOf()

}
{% endhighlight %}

The entity "Permission" will have an "ID", a "description" and a many-to-many relation with Role.

The code above will likely fail to create proper database structures. We must add some annotations to make it
clear to JPA that we want to create a "junction table".

First of all, we modify "Role" as follows:

{% highlight kotlin %}
@Entity
open class Role : Serializable
{
    @Id
    var id: Long = 0L
        private set

    @Column(unique = true)
    var description: String = ""

    @OneToMany(mappedBy = "role", cascade = arrayOf(CascadeType.ALL))
    var users: MutableSet<User> = mutableSetOf()

    @ManyToMany(fetch = FetchType.EAGER, cascade = arrayOf(CascadeType.MERGE))
    @JoinTable(name = "Role_Permissions",
            joinColumns = arrayOf(JoinColumn(name = "role_id")),
            inverseJoinColumns = arrayOf(JoinColumn(name = "permission_id")))
    var permissions: MutableSet<Permission> = mutableSetOf()

}
{% endhighlight %}

Notice how we modified the many-to-many attribute "permissions":

We defined that:
+ the JOIN operation is of type MERGE. We want that if the source entity is merged, 
the merge is cascaded to the target of the association.
+ when a row is called, all of its permissions are retrieved immediately (fetch EAGER)
+ the junction table is called "Role_Permissions" and it will be constituted by
a column with the ID of Role and a column with the ID of Permission.

Now we can create some Role with Permissions. Given that we already have a Permission table
with some values, this is the code that populates the Role table with some default permissions
(in Java this time):

{% highlight java %}
List<String> roles = new ArrayList<>();
roles.add("System Administrator");
roles.add("Power User");
roles.add("Analyst User");
roles.add("Business User");
roles.add("Guest");
Permission defaultPermission1 = this.permissions.findByDescription("read");
Permission defaultPermission2 = this.permissions.findByDescription("write");
Set<Permission> defaultPermissions = new HashSet<>();
defaultPermissions.add(defaultPermission1);
defaultPermissions.add(defaultPermission2);
for (String role : roles)
{
    Role tempRole = new Role();
    tempRole.setDescription(role);
    tempRole.setPermissions(defaultPermissions);
    this.roles.save(tempRole);
}
{% endhighlight %}

where "roles" object is the Spring Data [DAO] object that manages CRUD operations on the "Role" table.




[one-to-one]: http://www.objectdb.com/api/java/jpa/OneToOne
[one-to-many]: http://www.objectdb.com/api/java/jpa/OneToMany
[JPA]: https://docs.oracle.com/javaee/6/tutorial/doc/bnbpz.html
[Hibernate]: http://hibernate.org/
[Object-to-Relational-Mapper]: https://en.wikipedia.org/wiki/Object-relational_mapping
[Kotlin]: https://kotlinlang.org/
[DAO]: https://docs.spring.io/spring/docs/4.2.x/spring-framework-reference/html/dao.html
