require 'rake'
require 'rake/rdoctask'

# FIX: Hoe always calls rdoc with -d, and diagram generation fails here
class Rake::RDocTask
    alias __option_list__ option_list
    def option_list
	options = __option_list__
	options.delete("-d")
	options
    end
end

require './lib/utilrb/common'

begin
    require 'hoe'
rescue LoadError => e
    STDERR.puts "cannot load the Hoe gem. Distribution is disabled"
    STDERR.puts "error message is: #{e.message}"
end

begin
    hoe_spec = Hoe.spec 'utilrb' do
        developer "Sylvain Joyeux", "sylvain.joyeux@m4x.org"
        extra_deps <<
            ['facets', '>= 2.4.0'] <<
            ['rake', '>= 0']

        extra_dev_deps <<
            ['flexmock', '>= 0.8.6']

        self.summary = 'Yet another Ruby toolkit'
        self.description = paragraphs_of('README.txt', 3..5).join("\n\n")
    end
    hoe_spec.spec.extensions << 'ext/extconf.rb'

rescue Exception => e
    STDERR.puts "cannot load the Hoe gem, or Hoe fails. Distribution is disabled"
    STDERR.puts "error message is: #{e.message}"
end

RUBY = RbConfig::CONFIG['RUBY_INSTALL_NAME']
desc "builds Utilrb's C extension"
task :setup do
    Dir.chdir("ext") do
	if !system("#{RUBY} extconf.rb") || !system("make")
	    raise "cannot build the C extension"
	end
    end
    FileUtils.ln_sf "../ext/utilrb_ext.so", "lib/utilrb_ext.so"
end

Rake.clear_tasks(/publish_docs/)
task 'publish_docs' => 'redocs' do
    if !system('./update_github')
        raise "cannot update the gh-pages branch for GitHub"
    end
    if !system('git', 'push', 'github', '+gh-pages')
        raise "cannot push the documentation"
    end
end

task :clean do
    puts "Cleaning extension in ext/"
    FileUtils.rm_f "lib/utilrb_ext.so"
    if File.file?(File.join('ext', 'Makefile'))
        Dir.chdir("ext") do
            system("make clean")
        end
    end
    FileUtils.rm_f "ext/Makefile"
    FileUtils.rm_f "lib/utilrb_ext.so"
end

task :full_test do
    ENV['UTILRB_EXT_MODE'] = 'no'
    system("testrb test/")
    ENV['UTILRB_EXT_MODE'] = 'yes'
    system("testrb test/")
end

task :rcov_test do
    Dir.chdir('test') do 
	if !File.directory?('../rcov')
	    File.mkdir('../rcov')
	end
	File.open("../rcov/index.html", "w") do |index|
	    index.puts <<-EOF
		<!DOCTYPE html PUBLIC 
		    "-//W3C//DTD XHTML 1.0 Transitional//EN" 
		    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
		<body>
	    EOF

	    Dir.glob('test_*.rb').each do |path|
		puts "\n" * 4 + "=" * 5 + " #{path} " + "=" * 5 + "\n"
		basename = File.basename(path, '.rb')
		system("rcov --replace-progname -o ../rcov/#{basename} #{path}")
		index.puts "<div class=\"test\"><a href=\"#{basename}/index.html\">#{basename}</a></div>"
	    end
	    index.puts "</body>"
	end
    end
end

