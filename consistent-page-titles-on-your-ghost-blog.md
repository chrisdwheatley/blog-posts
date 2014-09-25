#### Here's a quick tip for fellow Ghost bloggers to help make your page titles more consistent and possibly more SEO friendly at the same time.

In the current version of Ghost (0.5.1) page titles are structured like so,

**Homepage:**

> Blog title (e.g. Chris Wheatley)

**Post:**

> Post title (e.g. Consistent Page Titles on Your Ghost Blog)

This feels a little disjointed. Personally I prefer something which offers more information to visitors, something along the lines of;

**Homepage:**

> Blog title - Blog description (e.g. Chris Wheatley - Software engineer surfing the front-end wave.)

**Post:**

> Post title - Blog title - Blog description (e.g. Consistent Page Titles on Your Ghost Blog - Chris Wheatley - Software engineer surfing the front-end wave.)

#### How can we achieve this?

To begin with, thanks to the power of open source software, I've submitted [an issue](https://github.com/TryGhost/Ghost/issues/4080) to the Ghost core library with my suggestions. Once I've received some feedback I plan to submit a pull request with the possibility that the change will be made in a future version of Ghost.

In the meantime however, you can make a quick edit to the `default.hbs` file which resides within the folder of the theme you're currently using, somewhere along the lines of `content/themes/{{theme-name}}/`.

Amend the `<title></title>` tags within `default.hbs` with the following piece of code and you're good to go.

```
<title>
  {{#if post.title}}
  	{{post.title}} - {{@blog.title}} - {{@blog.description}}
  {{else}}
  	{{@blog.title}} - {{@blog.description}}
  {{/if}}
</title>
```

You can always remove the blog description at the end of the title for something shorter or even add your own static text if you want.
