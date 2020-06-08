name: Why Cypress
class: middle, center, title

# Why Cypress!?
### Ryan Parsley
#### April 6, 2020

???

Hello, I'm Ryan and I want to tell you about Cypress today. If you're not
familiar, it's an E2E framework thats an alternative to Protractor. We're using
it on Carrier and it's great.

---
class: middle, center

# But Ryan, Angular uses Protractor!

???

This is not a big deal. As E2E tests depend on a working app, it's probably
simpler to replace the E2E tooling than unit test tooling (which we also did
about this time last year). A great thing about the Angular CLI is that while
it's opinionated, those opinions do not preclude your own. You can easily make
configuration changes to get cypress up and running.

---

background-image: url(./assets/yunoprotractor.jpg)
background-size: 500px auto

???

Ok, so you can. Why should you switch to Cypress?

---
# Why should I care?

* Interactive DOM selector
* Time travel debugger
* Console access during test runs
* Less Flaky
* Discoverable Best Practices

???

In a nutshell, it's a better developer experience across the board.

---

background-image: url(./assets/mock.jpg)
background-size: 500px auto

# Mock... Yeah!

???

This was _the_ killer feature for Carrier. Automated setup and tear down was off
the table for us and manual manipulation was untenable. Mocking enables us to
run more tests faster and with zero risk of unintentionally breaking something.

See also: Schrodinger's Bug

---

background-image: url(./assets/protractorDocs.png
background-size: 500px auto

# Protractor's documentation

???

An empty page with a link out to source code on github is not the level of
documentation that I'd expect from an established/ mature offering.

The landing page still has sample code from an angularjs app!

---

background-image: url(./assets/opinionatedDocs.png
background-size: 500px auto

# Docs are littered with best practices

???

This is the class of documentation that we need to help developers be
productive.

---

background-image: url(./assets/brokeDownDelorean.png
background-size: 450px auto

# Time travel debugging

???

Broken is the natural state of software development. Cypress provides better
tools for navigating these murky waters.

---

background-image: url(./assets/commandLog.png
background-size: 500px auto

# No, really!

???

You can walk through each command run over the course of the test run and
inspect state at any given point. This is invaluable! It sure beats the feedback
of "expected true to be false".

---
background-image: url(./assets/1999.jpg)
background-size: 500px auto

# Protractor: party like it's 1999

???

Protractor shows it's age when you think about how little it does to help you
with things like async events. Just thinking about a pattern called "page
object" feels not akin to an SPA

---

# Common protractor pitfall

```javascript
public async verifySearchButtonIsVisibleAfterScreenResize() {
  // this helper looks sensible, but it wraps sleep() :(
  await this.actions.shortWait('Short Wait');
  const width = 1000;
  const height = 800;
  browser.driver.manage().window().setSize(width, height);
  // YOLO: I guess short wasn't long enough?
  await this.actions.mediumWait('Medium Wait');
  await this.actions.moveMouseToElement(
    this.myLoadsSearchIcon,
    'Move mouse to Icon'
  );
}
```

???

Handling async is not intuitive. Devs fall in a pit of waiting amounts of time
and hoping for the best. Craig audited a real project that had 28minutes of
hard coded waiting in it.

---

# A better approach
## Resolve promises instead of choosing arbitrary times

```javascript
beforeEach(async () => {
  await myLoads.open('/my-loads?doAsUserID=JBHKOPA2');
  await pageStable();
});

const pageStable = async() => {
  Promise.all([
    await browser.wait(EC.not(EC.visibilityOf(myLoads.spinner)), 2000),
    await myLoads.actions.waitUntilElementInvisible(
      myLoads.growl,
      'Wait for growl to go away'
    )
  ]);
};

it('should search and handle no results gracefully', async() => {
  expect(myLoads.orderNumber.isDisplayed()).toBe(false);
  await myLoads.tableSearchToggle.click();
  await myLoads.orderNumber.sendKeys('foo');
  await myLoads.tableSearchSubmit.click();
  expect(myLoads.tableNoResults.isDisplayed()).toBe(true);
});
```

???

I teach the above as a best practice for waiting. It's less flaky than the
previous code example, but it's predicated on a bit of understanding of how
your app works that feels onerous.

---

# Even better approach
## use a framework that doesn't need all that cruft

```
it('should handle no results gracefully', function() {
  cy.visit('http://localhost:4200/360/ngx/my-loads')
  cy.get('.ui-table-caption > .icon-Search').click()
  cy.get('#orderNumber').type('foo')
  cy.get('p-footer > .ui-button > .ui-button-text').click()
  cy.get('.empty-message').should('be.visible')
})
```

???

Cypress implicitly polls and waits for things to resolve with sensible
timeouts. You can customize and establish one off exceptions... or just let the
defaults be sensible.

---

# Selectors
## Copy paste from chrome inspector is not a good strategy

```javascript
assignDriverButtonInCTA = element(
  by.className(
    `button-margins jb-dark-background-button-primary 
     ui-button ui-widget ui-state-default ui-corner-all
     ui-button-text-empty ng-star-inserted`
  )
);
```

???

This is a real life example of what developers do left to their own devices when
writing E2E tests

---

background-image: url(./assets/playground.gif)
background-size: 500px auto

#Selector playground

???

Armed with knowledge of how your app is constructed you can customize selector
priority so the recommendations are tailored to your preferences.

---

# What's CI look like?

```javascript
// package.json
...
scripts: {
  ...
  "ci": "run-s ci:lint ci:build ci:cy-run",
  "ci:lint": "ng lint",
  "ci:build": "ng build --configuration cypress",
  "ci:start-server": "angular-http-server --silent --path ./dist -p 4200",
  "ci:cy-run":
    "start-server-and-test ci:start-server http://localhost:4200 cy:run",
}
...

```

---

# Links

* [Cypress Docs](https://www.cypress.io/)
* [Benefits of Cypress documentation I put together](https://jbhunt.visualstudio.com/EngAndTech/_wiki/wikis/Applications.wiki/860/Benefits-of-Cypress?anchor=use-a-framework-that-doesn%27t-need-all-that-cruft)
* [Guide to setting up Angular with Cypress](https://www.cypress.io/blog/2019/08/02/guest-post-angular-adding-cypress-ui-tests-to-your-devops-pipeline/)
* [ng-conf Cypress workshop](https://app.pluralsight.com/player?course=ng-conf-19-testing-cypress-io&author=ng-conf&name=6bc8bea6-1905-4595-8fc8-4c4426e627c6&clip=0&mode=live)
