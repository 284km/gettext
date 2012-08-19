# -*- ruby -*-
#
# Rakefile for Ruby-GetText-Package
#
# This file maintains Ruby-GetText-Package.
#
# Use setup.rb or gem for installation.
# You don't need to use this file directly.
#
# Copyright(c) 2005-2009 Masao Mutoh
# Copyright(c) 2012 Kouhei Sutou <kou@clear-code.com>
# Copyright(c) 2012 Haruka Yoshihara <yoshihara@clear-code.com>
# This program is licenced under the same licence as Ruby.

base_dir = File.expand_path(File.dirname(__FILE__))
$LOAD_PATH.unshift(File.join(base_dir, 'lib'))

require "tempfile"
require "rake"
require "rubygems"
require "yard"
require "gettext/version"
require "gettext/tools"
require "gettext/task"
require "bundler/gem_helper"

class Bundler::GemHelper
  undef_method :version_tag
  def version_tag
    version
  end
end

helper = Bundler::GemHelper.new(base_dir)
helper.install
spec = helper.gemspec

PKG_VERSION = GetText::VERSION

task :default => :test

############################################################
# GetText tasks for developing
############################################################
poparser_rb_path = "lib/gettext/tools/poparser.rb"
desc "Create #{poparser_rb_path}"
task :poparser => poparser_rb_path

poparser_ry_path = "src/poparser.ry"
file poparser_rb_path => poparser_ry_path do
  racc = File.join(Gem.bindir, "racc")
  tempfile = Tempfile.new("gettext-poparser")
  command_line = "#{racc} -g #{poparser_ry_path} -o #{tempfile.path}"
  ruby(command_line)
  $stderr.puts("ruby #{command_line}")

  File.open(poparser_rb_path, "w") do |poparser_rb|
    poparser_rb.puts(<<-EOH)
# -*- coding: utf-8 -*-
#
# poparser.rb - Generate a .mo
#
# Copyright (C) 2003-2009 Masao Mutoh <mutomasa at gmail.com>
# Copyright (C) 2012 Kouhei Sutou <kou@clear-code.com>
#
# You may redistribute it and/or modify it under the same
# license terms as Ruby or LGPL.

EOH

    poparser_rb.puts(tempfile.read)
  end
  $stderr.puts "Create #{poparser_rb_path}."
end


GetText::Task.new(spec)
Dir.glob("samples/*.rb") do |target|
  domain = File.basename(target, ".*")
  GetText::Task.new(spec) do |task|
    task.domain = domain
    task.namespace_prefix = "samples:#{domain}"
    task.po_base_directory = "samples/po"
    task.mo_base_directory = "samples"
    task.files = Dir.glob(target.gsub(/\.*+\z/, ".*"))
  end
  task "samples:gettext" => "samples:#{domain}:gettext"
end
desc "Update *.mo for samples"
task "samples:gettext"

[
  ["main", Dir.glob("samples/cgi/{index.cgi,cookie.cgi}")],
  ["helloerb1", Dir.glob("samples/cgi/helloerb1.cgi")],
  ["helloerb2", Dir.glob("samples/cgi/helloerb2.cgi")],
  ["hellolib", Dir.glob("samples/cgi/hellolib.rb")],
].each do |domain, files|
  GetText::Task.new(spec) do |task|
    task.domain = domain
    task.namespace_prefix = "samples:cgi:#{domain}"
    task.po_base_directory = "samples/cgi/po"
    task.mo_base_directory = "samples/cgi"
    task.files = files
  end
  task "samples:cgi:gettext" => "samples:cgi:#{domain}:gettext"
end
desc "Updates *.mo for CGI samples"
task "samples:cgi:gettext"

task "samples:gettext" => "samples:cgi:gettext"

["backslash", "non_ascii", "np_", "ns_", "p_", "s_"].each do |domain|
  GetText::Task.new(spec) do |task|
    task.domain = domain
    task.namespace_prefix = "test:#{domain}"
    task.po_base_directory = "test/po"
    task.mo_base_directory = "test"
    task.files = ["test/fixtures/#{domain}.rb"]
    task.locales = ["ja"]
  end
  task "test:gettext" => "test:#{domain}:gettext"
end

["_"].each do |domain|
  GetText::Task.new(spec) do |task|
    task.domain = domain
    task.namespace_prefix = "test:#{domain}"
    task.po_base_directory = "test/po"
    task.mo_base_directory = "test"
    task.files = ["test/fixtures/#{domain}.rb"]
    task.files += Dir.glob("test/fixtures/#{domain}/*.rb")
    task.locales = ["ja"]
  end
  task "test:gettext" => "test:#{domain}:gettext"
end

["plural"].each do |domain|
  GetText::Task.new(spec) do |task|
    task.domain = domain
    task.namespace_prefix = "test:#{domain}"
    task.po_base_directory = "test/po"
    task.mo_base_directory = "test"
    task.files = []
  end
  task "test:gettext" => "test:#{domain}:gettext"
end
desc "Update *.mo for test"
task "test:gettext"


task :package => [:makemo]

desc "Run all tests"
task :test => "test:prepare" do
  options = ARGV - Rake.application.top_level_tasks
  ruby "test/run-test.rb", *options
end

namespace :test do
  desc "Prepare test environment"
  task :prepare => ["test:gettext", "samples:gettext"]
end

YARD::Rake::YardocTask.new do |t|
end

desc "Setup Ruby-GetText-Package. (for setup.rb)"
task :setup => [:makemo]
