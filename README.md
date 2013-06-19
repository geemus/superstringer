# superstringer

Automation for managing [stringer](https://github.com/swanson/stringer) on [Heroku](https://heroku.com).

NOTE: superstringer uses netrc for authentication. [Heroku Toolbelt](https://toolbelt.heroku.com) will automatically configure this when you login, so you can download/install and run `heroku login` to generate this file if needed.

## Deploying to Heroku

```term
git clone git://github.com/geemus/superstringer
cd superstringer
bundle install
bundle exec rake heroku:deploy
```

On completion you should see instructions for adding a rake task in scheduler to update your feeds and a link to your installation. For further info see [stringer](https://github.com/swanson/stringer).
