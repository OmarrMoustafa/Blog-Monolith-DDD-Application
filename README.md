# Blog Monolith DDD.

Standalone API project that can be referred to as a headless blog, which is a fancy term to say that this is only a backend platform without a UI. Multiple UIs can interact with it, so it is not coupled to one UI.

In DDD terms, the
controller was acting as an application service.

## The Application Project

Our solution communicates with the outside world, the clients, and the Uqs.Blog.WebApi project. This solution is using a RESTful Web API to communicate with the UI layer. This should make it easy for a browser-based UI layer, such as React, Angular, Vue, or Blazor, to exchange the data (in the form of contracts).

## The contract objects project

To communicate with the outside world, your data should have a defined structure. This defined structure comprises classes, and it is in the **Uqs.Blog.Contract** project.

If this is a UI project, these contracts might be called **view models**, as they are models that are *bound* to the UI (the view) directly. Also, they might be called **data transportation objects** (**DTOs**) as they transport the data from the server to the clients. In API projects, they are usually referred to as **contracts**.

Put simply, if the RESTful APIs have an API that requests the full information of a post, with the following URL:

```csharp
https://api.uqsblog/posts/1
```

Then the contract might look like this:

```csharp
public record Author(int Id, string Name);
public record Post(int Id, string Content, Author Author,
    DateTime CreatedDate, int NumberOfComments
    , int NumberOfViews, …);
```

This is usually transferred in JSON.

Contracts are not part of the DDD philosophy, but they are needed here to have a complete application.

## The domain layer project

The components of this layer are in **Uqs.Blog.Domain**. This is where all the types related to the domain design live.

Consider my approach here as *an example* rather than *the way* to do it.

This layer contains the following:

- Business logic
- Database persistence

## Domain Objects

A **domain object** is a representation of a real-life business entity.

These entities generally map directly to relational database tables, so you have database tables for posts, authors, tags, and so on.

In a **document database**, the domain objects may or may not be the ones persisted directly into your collection.

DDD differentiates between two types of domain objects: entities and value objects.

In our preceding blog example, **Post**, **Author**, **Comment**, and **Commenter** are entities. **Tag** is value object.

## Classes

#### Aggregate

The previous blog classes set a distinguished business objective, which is managing a blog post. These classes form an aggregate.

#### Anemic

In an anemic model, the client interprets the purpose and use of the domain object, and the business logic ends up being implemented in other classes. An anemic model is considered an anti-pattern, as it opposes the theories of OOP.

Obviously, we have not followed the DDD recommendation in our implementation and wrote the **UpdateTitle** method in the service class.

## Domain Services

Services in DDD are divided into three types: infrastructure services and application services.

#### Adding Post Service

Adding a new post will require the author’s ID but no other field. The code for this service can look as such:

```csharp
public class AddPostService
{
    private readonly IPostRepository _postRepository;
    private readonly IAuthorRepository _authorRepository;
    public AddPostService(IPostRepository postRepository,
         IAuthorRepository authorRepository)
    {
        _postRepository = postRepository;
        _authorRepository = authorRepository;
    }

    public int AddPost(int authorId)
    {
        var author = _authorRepository.GetById(authorId);
        if (author is null)
        {
            throw new ArgumentException(
              "Author Id not found",nameof(authorId));
        }
        if (author.IsLocked)
        {
            throw new InvalidOperationException(
              "Author is locked");
        }
        var newPostId = _postRepository.CreatePost
          (authorId);
        return newPostId;
    }
}
```

First, you’ll notice that I have dedicated a whole class, **AddPostService**, with a single method, **AddPost**. Some designs create a single service class such as **PostService** and add multiple business logic methods inside it. I opted for the single public method in a single class approach to respect the single-responsibility principle of **SOLID**.

#### Updating title service

The title of the blog can be up to 90 characters and can be updated at any time. This is sample code to achieve this:

```csharp
public class UpdateTitleService
{
    private readonly IPostRepository _postRepository;
    private const int TITLE_MAX_LENGTH = 90;
    public UpdateTitleService(IPostRepository postRepo)
    {
        _postRepository = postRepo;
    }
    public void UpdateTitle(int postId, string title)
    {
        if (title is null) title = string.Empty;
        title = title.Trim();
        if (title.Length > TITLE_MAX_LENGTH)
        {
            throw new
               ArgumentOutOfRangeException(nameof(title),
               $"Title max is {TITLE_MAX_LENGTH} letters");
        }
        var post = _postRepository.GetById(postId);
        if (post is null)
        {
            throw new ArgumentException(
                $"Unable to find a post of Id {postId}",
                nameof(post));
        }
        post.Title = title;
        _postRepository.Update(post);
    }
}
```

The preceding logic is straightforward.

In both services, the business logic involved no knowledge of the data platform. This can be SQL Server, Cosmos DB, MongoDB, and so on. DDD refers to the libraries for these tools as infrastructure, so the services have no knowledge of the infrastructure.

## Repositories

Repositories are a way of achieving a single responsibility (as in SOLID’s single responsibility principle) by having the services and the domains responsible for business logic but not responsible for data persistence. DDD gives the data persistence responsibility to the repositories.

## Architectural View

![Architectural View](https://github.com/OmarrMoustafa/Blog-Monolith-DDD-Application/assets/120117521/5a53ec67-78f2-48e0-ac1e-5bf901bbc3a9)
