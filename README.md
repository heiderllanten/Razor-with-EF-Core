# Razor Pages with Entity Framework Core in ASP.NET Core

## The data model

### Relationships

A relationship defines how two entities relate to each other. In a relational database, this is represented by a foreign key constraint.

**Navigation property:** Navigation properties hold other entities that are related to this entity. A property defined on the principal and/or dependent entity that references the related entity.

The Grade property is an enum. The question mark after the Grade type declaration indicates that the Grade property is nullable. A grade that's null is different from a zero grade—null means a grade isn't known or hasn't been assigned yet.