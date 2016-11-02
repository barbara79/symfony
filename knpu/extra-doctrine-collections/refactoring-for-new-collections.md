# Using the new OneToMany Collections

Open up the `genus/show.html.twig` template. Actually, let's start in the `Genus`
class itself. Find `getGenusScientist()`. This method is lying! It doesn't return
an array of `User` objects, it returns an array of `GenusScientist` objects!

In the template, when we loop over `genus.genusScientist`, `genusScientist` is *not*
a `User` anymore. Update to `genusScientist.user.fullName`, and above, for the `user_show`
route, change this to `genusScientist.user.id`. 

Then, in the link, let's show off our new `yearsStudied` field:
`{{ genusScientist.yearStudied }}` then years. We still need to fix the remove link,
but let's see how it looks so far!

Refresh! Ok, no more errors. Well, until you click to view the user!

## Updating the User Template

Start by opening `User` and finding `getStudyGenuses()`. Change the PHPDoc to advertise
that this *now* returns an array of `GenusScientist` objects.

Now let's go fix the template: `user/show.html.twig`. Hmm, let's rename this variable
to be a bit more clear: `genusScientist`, to match the type of object it is. Now,
update `slug` to be `genusScientist.genus.slug`. And print `genusScientist.genus.name`.

Try it! Page back to life!

## Updating the Delete Link

Back on the genus page, the other thing we need to fix is this remove link. In the
`show.html.twig` template for genus, update the `userId` part of the URL:
`genusScientist.user.id`.

Next, find this endpoint in `GenusController`: `removeGenusScientistAction`. It's
about to get *way* nicer. Kill the queries for `Genus` and `User`. Replace them with
`$genusScientist = $em->getRepository('AppBundle:GenusScientist')` and, `findOneBy`,
passing it `user` set to `$userId` and `genus` set to `$genusId`.

Then, instead of removing this link from `Genus`, we simply delete the entity:
`$em->remove($genusScientist)`. And celebrate!

Go try it! Quick, delete that scientist! It disappears in dramatic fashion, *and*,
when we refresh, it's *definitely* gone.

Phew! We're almost done. By the way, you can see that this is a good amount of work.
If you know that your join table will probably need extra fields on it, you can save
yourself this work by setting up the join entity from the very beginning and avoiding
`ManyToMany`. But, if you definitely won't have extra fields, `ManyToMany` is way
nicer.

## Updating the Fixtures

The *last* thing to fix is the fixtures. We won't set the `genusScientists` property
up here anymore. Instead, scroll down and add a new `AppBundle\Entity\GenusScientist`
section. It's simple: we'll just build new `GenusScientist` objects ourselves, just
like we did in `newAction()` in PHP code a minute ago. Add `genus.scientist_{1..50}`
to create 50 links. Then, assign `user` to a random `@user.aquanaut_*` and `genus`
to a random `@genus_*`. And hey, set `yearsStudied` to something random too:
`<numberBetween(1, 30>`.

Yes! Go find your terminal and reload!

```bash
php bin/console doctrine:fixtures:load
```

Ok, go back to `/genus`... and click one of them. We have scientists!

So our app is fixed, right? Well, not so fast. Go to `/admin/genus`: you might need
to log back in - password `iliketurtles`. Our genus form is still *totally* broken.
Ok, not error: but it doesn't even make sense anymore: our relationships is now more
complex than checkboxes can handle. How would I set the `yearsStudied`? Time to take
this form up a level.