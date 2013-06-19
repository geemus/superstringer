require "bundler"
Bundler.require

require "fileutils"
require "json"
require "securerandom"
require "tmpdir"

namespace :heroku do

  desc "deploy stringer on Heroku"
  task :deploy do
    Formatador.display_line("[negative]<> deploying stringer to Heroku[/]")

    # grab netrc credentials, set by toolbelt via `heroku login`
    _, password = Netrc.read['api.heroku.com']

    # setup excon for API calls
    heroku = Excon.new(
      'https://api.heroku.com',
      :headers => {
        "Accept"        => "application/vnd.heroku+json; version=3",
        "Authorization" => "Basic #{[':' << password].pack('m').delete("\r\n")}",
        "Content-Type"  => "application/json"
      }
    )

    # git clone git://github.com/swanson/stringer.git
    tmpdir = Dir.mktmpdir
    Formatador.display_line("[negative]<> cloning code to [underline]#{tmpdir}[/]")
    FileUtils.chdir(tmpdir) do
      `git clone git://github.com/swanson/stringer.git`
    end

    #cd stringer
    FileUtils.chdir(File.join(tmpdir, 'stringer')) do
      #heroku create
      Formatador.display_line("[negative]<> creating app[/]")
      app_data = JSON.parse(heroku.post(:path => "/apps").body)
      `git remote add heroku #{app_data['git_url']}`

      #git push heroku master
      Formatador.display_line("[negative]<> pushing code to [underline]#{app_data['name']}[/]")
      `git push heroku master`

      heroku.reset # reset socket as git push may take long enough for timeout

      #heroku config:set SECRET_TOKEN=`openssl rand -hex 20`
      Formatador.display_line("[negative]<> setting SECRET_TOKEN on [underline]#{app_data['name']}[/]")
      heroku.patch(
        :body       => { "SECRET_TOKEN" => SecureRandom.hex(20) }.to_json,
        :path       => "/apps/#{app_data['id']}/config-vars"
      )

      #heroku run rake db:migrate
      Formatador.display_line("[negative]<> running `rake db:migrate` on [underline]#{app_data['name']}[/]")
      run_data = JSON.parse(heroku.post(
        :body => {
          "attach"  => true,
          "command" => "rake db:migrate"
        }.to_json,
        :path => "/apps/#{app_data['id']}/dynos"
      ).body)
      Rendezvous.start(
        :url => run_data['attach_url']
      )

      heroku.reset # reset socket as db:migrate may take long enough for timeout

      #heroku restart
      Formatador.display_line("[negative]<> restarting [underline]#{app_data['name']}[/]")
      heroku.delete(:path => "/apps/#{app_data['id']}/dynos")

      #heroku addons:add scheduler
      Formatador.display_line("[negative]<> adding scheduler:standard to [underline]#{app_data['name']}[/]")
      heroku.post(
        :body => { "plan" => { "name" => "scheduler:standard" } }.to_json,
        :path => "/apps/#{app_data['id']}/addons"
      )

      #heroku addons:open scheduler
      Formatador.display_lines([
        "[negative]<> Add `[bold]rake fetch_feed[/][negative]` hourly task at [underline]https://api.heroku.com/apps/#{app_data['id']}/addons/scheduler:standard[/]",
        "[negative]<> stringer available at [underline]#{app_data['web_url']}[/]"
      ])
    end
  end

  desc "update stringer on heroku"
  task :update do |app|
    # TODO: exercise for reader
  end

end
