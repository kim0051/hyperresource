require 'rake/testtask'
require './lib/hyper_resource/version'

task :default => [:test]

Rake::TestTask.new do |t|
  t.libs << 'test'
  t.libs << 'test/lib'
  t.test_files = FileList['test/unit/*_test.rb'] +
                 FileList['test/live/*_test.rb']
  t.warning = !!ENV['WARNINGS']
  t.verbose = true
end

task :release do
  system(<<-EOT)
    git add lib/hyper_resource/version.rb
    git commit -m 'release v#{HyperResource::VERSION}'
    git push origin
    git tag v#{HyperResource::VERSION}
    git push --tags origin
    gem build hyper_resource.gemspec
    gem push hyperresource-#{HyperResource::VERSION}.gem
  EOT
end

task :test_server do
  require './test/live/live_test_server'
  port = ENV['PORT'] || ENV['port'] || 3000
  Rack::Handler::WEBrick.run(LiveTestServer.new, :Port => port)
end

