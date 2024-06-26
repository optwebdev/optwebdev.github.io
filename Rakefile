# encoding: UTF-8
require "rubygems"
require "tmpdir"
require "bundler/setup"
require "jekyll"
require 'digest/md5'
require 'RMagick'
require 'open-uri'

# Change your GitHub reponame
GITHUB_REPONAME    = "optweb/optweb.github.io"
GITHUB_REPO_BRANCH = "main"

SOURCE = ""
DEST   = "_site"
CONFIG = {
  'layouts' => "_layouts",
  'posts' => "_posts",
  'post_ext' => "md",
  'categories' => "categories",
  'tags' => "tags"
}

task default: %w[publish]

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "destination" => "_site",
    "config"      => "_config.yml"
  })).process
end

desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    cp_r "_site/.", tmp

    pwd = Dir.pwd
    Dir.chdir tmp

    system "git init"
    system "git checkout --orphan #{GITHUB_REPO_BRANCH}"
    system "git add ."
    message = "Site updated at #{Time.now.utc}"
    system "git commit -am #{message.inspect}"
    system "git remote add origin https://github.com/#{GITHUB_REPONAME}"
    system "git push origin #{GITHUB_REPO_BRANCH} --force"

    Dir.chdir pwd
  end
end

desc "Begin a new post in #{CONFIG['posts']}"
task :post do
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
  title = ENV["title"] || "Novo post"

  tags = ""
  categories = ""

  # tags
  env_tags = ENV["tags"] || ""
  tags = strtag(env_tags)

  # categorias
  env_cat = ENV["category"] || ""
  categories = strtag(env_cat)

  # slug do post
  slug = mount_slug(title)

  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
    time = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%T')
  rescue => e
    puts "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end

  filename = File.join(CONFIG['posts'], "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: post"
    post.puts "title: \"#{title.gsub(/-/,' ')}\""
    # post.puts "permalink: #{slug}"
    post.puts "date: #{date} #{time}"
    post.puts "comments: true"
    post.puts "description: \"#{title}\""
    post.puts 'keywords: ""'
    post.puts "categories:"
    post.puts "#{categories}"
    post.puts "tags:"
    post.puts "#{tags}"
    post.puts "---"
  end
end # task :post


desc "Create a new page."
task :page do
  name = ENV["name"] || "new-page.md"
  filename = "#{name}"
  filename = File.join(filename, "index.html") if File.extname(filename) == ""
  title = File.basename(filename, File.extname(filename)).gsub(/[\W\_]/, " ").gsub(/\b\w/){$&.upcase}
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  slug = mount_slug(title)

  mkdir_p File.dirname(filename)
  puts "Creating new page: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "layout: page"
    post.puts "title: \"#{title}\""
    post.puts 'description: ""'
    post.puts 'keywords: ""'
    # post.puts "permalink: \"#{slug}\""
    post.puts "slug: \"#{slug}\""
    post.puts "---"
    post.puts "{% include JB/setup %}"
  end
end # task :page

def mount_slug(title)
  slug = str_clean(title)
  slug = slug.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')

  return slug
end

def str_clean(title)
  return title.tr("ÀÁÂÃÄÅàáâãäåĀāĂăĄąÇçĆćĈĉĊċČčÐðĎďĐđÈÉÊËèéêëĒēĔĕĖėĘęĚěĜĝĞğĠġĢģĤĥĦħÌÍÎÏìíîïĨĩĪīĬĭĮįİıĴĵĶķĸĹĺĻļĽľĿŀŁłÑñŃńŅņŇňŉŊŋÒÓÔÕÖØòóôõöøŌōŎŏŐőŔŕŖŗŘřŚśŜŝŞşŠšſŢţŤťŦŧÙÚÛÜùúûüŨũŪūŬŭŮůŰűŲųŴŵÝýÿŶŷŸŹźŻżŽž", "AAAAAAaaaaaaAaAaAaCcCcCcCcCcDdDdDdEEEEeeeeEeEeEeEeEeGgGgGgGgHhHhIIIIiiiiIiIiIiIiIiJjKkkLlLlLlLlLlNnNnNnNnnNnOOOOOOooooooOoOoOoRrRrRrSsSsSsSssTtTtTtUUUUuuuuUuUuUuUuUuUuWwYyyYyYZzZzZz")
end

def strtag(str_tags)
  tags = "";

  if !str_tags.nil?
    arr_tags = str_tags.split(",")
    arr_tags.each do |t|
      tags = tags + "- " + t.delete(' ') + "\n"
    end
  end

  return tags
end


# Automate favicon generation
# http://sarahcassady.com/2015/07/19/jekyll-gravatars-and-favicons/
namespace :assets do
  desc "Generate favicons using your Gravatar"
  task :favicons do
    config = YAML.load_file('_config.yml')
    gravatar_email = config['email']
    asset_path = 'images'
    name_pre = "favicon-%d.png"

    puts "Assembling Gravatar URL..."
    gravatar_hash = Digest::MD5.hexdigest(gravatar_email)
    gravatar_url = "https://www.gravatar.com/avatar/#{gravatar_hash}?s=192"

    origin = "origin.png"
    File.delete origin if File.exist? origin

    puts "Fetching Gravatar image..."
    open(origin, 'wb') do |file|
      file << open(gravatar_url).read
    end

    FileList["*apple-touch-ico*.png"].each do |img|
      File.delete img
    end

    FileList["*favicon.ico"].each do |img|
      File.delete img
    end

    [180, 152, 76, 120, 60, 144, 72, 114, 57, 64, 96, 160, 192, 32].each do |size|
      im = Magick::Image::read(origin).first.resize(size, size)
      circle = Magick::Image.new size, size
      gc = Magick::Draw.new
      gc.fill 'black'
      gc.circle size/2, size/2, size/2, 1
      gc.draw circle
      mask = circle.blur_image(0,1).negate
      mask.matte = false
      im.matte = true
      im.composite!(mask, Magick::CenterGravity, Magick::CopyOpacityCompositeOp)

      puts "Generating #{name_pre} icon..." % [size, size]
      im.write "#{asset_path}/" + name_pre % [size, size]
      if size == 32
        puts "Generating favicon.ico..."
        im.write "#{asset_path}/favicon.ico"
      end
    end

    puts "Cleaning up..."
    File.delete origin
  end
end
