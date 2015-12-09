Django class choice field
=========================

Django class choice field is a utility library designed to make it easy to use
one of a choice of classes as a field on a model. It can be used to encapsulate
logic associated with a type of the model, without using a polymorphic
solution.

```
from django.db import models
from class_choice_field import ClassChoiceField

class BlogPost(models.Model):
    title = models.CharField()
    status = ClassChoiceField([
        'blog.statuses.Draft',
        'blog.statuses.Approval',
        'blog.statuses.Live',
    ])
```

Rationale
---------

A common pattern for dealing with having different "flavours" of a model
instance is to specify the choices as class attributes and then treat code
conditionally depending on the value of the field.

```
from django.db import models

class BlogPost(models.Model):
    STATUS_DRAFT = 'draft'
    STATUS_APPROVAL = 'approval'
    STATUS_LIVE = 'live'
    STATUS_CHOICES = (
        (STATUS_DRAFT, 'Draft'),
        (STATUS_APPROVAL, 'Approval'),
        (STATUS_LIVE, 'Live'),
    )
    title = models.CharField()
    status = models.CharField(choices=STATUS_CHOICES)
```

This pattern is great when you have a small number of simple places where the
logic changes depending on the status of the object. However on occasions you
can have a lot of variation in the associated logic. In the example above, the
status could be used to change the rendered template, the associated edit form,
the permissions of who can view the object, the permissions of who can edit the
object… the list goes on. Instead of littering the code base with switches
based on the status, we can instead encapsulate all that logic into a `Status`
object and choose between them.

```
class Draft(ClassChoice):
    ...

class BlogPost(models.Model):
    title = models.CharField()
    status = ClassChoiceField([Draft])

    objects = models.Manager.from_queryset(status.queryset)

BlogPost.objects.filter(status='draft')
BlogPost.objects.filter(status=Draft)
BlogPost.objects.filter_draft()

post = BlogPost.objects.filter_draft()[0]
post.is_draft
post.status == 'draft'
post.status == Draft

post.status.template_name
post.status.form_class
post.status.has_edit_permission(user)
```
