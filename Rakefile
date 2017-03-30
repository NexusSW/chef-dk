#
# Copyright:: Copyright (c) 2016 Chef Software Inc.
# License:: Apache License, Version 2.0
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

require "bundler/gem_tasks"
require_relative "tasks/version"
require_relative "tasks/bundle"
require_relative "tasks/dependencies"
require_relative "tasks/github_changelog_generator"
require_relative "tasks/announce"

desc "Keep the Dockerfile up-to-date"
task :update_dockerfile do
  require "mixlib/install"
  latest_stable_version = Mixlib::Install.available_versions("chefdk", "stable").last
  text = File.read("Dockerfile")
  new_text = text.gsub(/^ARG VERSION=[\d\.]+$/, "ARG VERSION=#{latest_stable_version}")
  File.open("Dockerfile", "w+") { |f| f.write(new_text) }
end

namespace :style do
  begin
    require "rubocop/rake_task"

    desc "Run Cookbook Ruby style checks"
    RuboCop::RakeTask.new(:cookstyle) do |t|
      t.requires = ["cookstyle"]
      t.patterns = ["lib/chef-dk/skeletons/code_generator"]
      t.options = ["--display-cop-names"]
    end
  rescue LoadError => e
    puts ">>> Gem load error: #{e}, omitting #{task.name}" unless ENV["CI"]
  end

  begin
    require "rubocop/rake_task"

    ignore_dirs = Regexp.union(%w{
      lib/chef-dk/skeletons/code_generator
      spec/unit/fixtures/chef-runner-cookbooks
      spec/unit/fixtures/cookbook_cache
      spec/unit/fixtures/example_cookbook
      spec/unit/fixtures/example_cookbook_metadata_json_only
      spec/unit/fixtures/example_cookbook_no_metadata
      spec/unit/fixtures/local_path_cookbooks
    })

    desc "Run Chef Ruby style checks"
    RuboCop::RakeTask.new(:chefstyle) do |t|
      t.requires = ["chefstyle"]
      t.patterns = `rubocop --list-target-files`.split("\n").reject { |f| f =~ ignore_dirs }
      t.options = ["--display-cop-names"]
    end
  rescue LoadError => e
    puts ">>> Gem load error: #{e}, omitting #{task.name}" unless ENV["CI"]
  end

  begin
    require "foodcritic"

    desc "Run Chef Cookbook (Foodcritic) style checks"
    FoodCritic::Rake::LintTask.new(:foodcritic) do |t|
      t.options = {
        fail_tags: ["any"],
        tags: ["~FC011", "~FC064", "~FC065", "~FC066", "~FC067"],
        cookbook_paths: ["lib/chef-dk/skeletons/code_generator"],
        progress: true,
      }
    end
  rescue LoadError => e
    puts ">>> Gem load error: #{e}, omitting #{task.name}" unless ENV["CI"]
  end
end
