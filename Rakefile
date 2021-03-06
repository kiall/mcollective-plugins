require 'rake/rdoctask'
require 'rake/packagetask'
require 'rake/clean'
require 'find'

# Per Project Settings
 
PROJ_NAME = "mcollective-plugins"
PROJ_URL = "http://marionette-collective.org/"
PROJ_DOC_TITLE = "MCollective Plugins"
PROJ_VERSION = "1.0"
PROJ_RELEASE = "1"

PROJ_FILES = ["agent", "audit", "ext", "facts", "README.markdown", "registration", "security", "simplerpc_authorization", "utilities"]

PKG_MAINT_NAME = "Kiall Mac Innes"
PKG_MAINT_EMAIL = "kiall@managedit.ie"

# End Per Project Settings

ENV['DEB_RELEASE_TO'] ? PKG_DEB_DUPLOAD = ENV['DEB_RELEASE_TO'] : PKG_DEB_DUPLOAD = nil
ENV['DEB_DISTRIBUTION'] ? PKG_DEB_DISTRIBUTION = ENV['DEB_DISTRIBUTION'] : PKG_DEB_DISTRIBUTION = "unstable"
ENV["BUILD_NUMBER"] ? CURRENT_RELEASE = ENV["BUILD_NUMBER"] : CURRENT_RELEASE = PROJ_RELEASE
ENV["PKG_VERSION"] ? CURRENT_VERSION = ENV["PKG_VERSION"] : CURRENT_VERSION = PROJ_VERSION

CLEAN.include(["build", "doc"])

def announce(msg='')
    STDERR.puts "================"
    STDERR.puts msg
    STDERR.puts "================"
end

def init
    FileUtils.mkdir("build") unless File.exist?("build")
end

def safe_system *args
    raise RuntimeError, "Failed: #{args.join(' ')}" unless system *args
end

desc "Build documentation, tar balls and rpms"
task :default => [:clean, :doc, :package]

# task for building docs
rd = Rake::RDocTask.new(:doc) { |rdoc|
    rdoc.rdoc_dir = 'doc'
    rdoc.template = 'html'
    rdoc.title    = "#{PROJ_DOC_TITLE} version #{CURRENT_VERSION}"
    rdoc.options << '--line-numbers' << '--inline-source' << '--main' << 'MCollective'
}

desc "Run spec tests"
task :test do
#    sh "cd spec;rake"
end

desc "Create a tarball for this release"
task :package => [:clean, :doc] do
    announce "Creating #{PROJ_NAME}-#{CURRENT_VERSION}.tgz"

    FileUtils.mkdir_p("build/#{PROJ_NAME}-#{CURRENT_VERSION}")
    safe_system("cp -R #{PROJ_FILES.join(' ')} build/#{PROJ_NAME}-#{CURRENT_VERSION}")

    announce "Setting Version to #{CURRENT_VERSION}"
    safe_system("cd build/#{PROJ_NAME}-#{CURRENT_VERSION} && find ./ -type f -exec sed -i 's/@DEVELOPMENT_VERSION@/#{CURRENT_VERSION}/' {} \\;")

    safe_system("cd build && tar --exclude .svn -cvzf #{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{PROJ_NAME}-#{CURRENT_VERSION}")
end

namespace :package do
    desc "Create .deb packages"
    task :deb => [:clean, :doc, :package] do
        announce("Building debian packages")

        FileUtils.mkdir_p("build/deb")
        Dir.chdir("build/deb") do
            safe_system %{tar -xzf ../#{PROJ_NAME}-#{CURRENT_VERSION}.tgz}
            safe_system %{cp ../#{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{PROJ_NAME}_#{CURRENT_VERSION}.orig.tar.gz}

            Dir.chdir("#{PROJ_NAME}-#{CURRENT_VERSION}") do
                safe_system %{cp -R ext/debian .}

                File.open("debian/changelog", "w") do |f|
                    f.puts("#{PROJ_NAME} (#{CURRENT_VERSION}-#{CURRENT_RELEASE}) #{PKG_DEB_DISTRIBUTION}; urgency=low")
                    f.puts
                    f.puts("  * Automated release for #{CURRENT_VERSION}-#{CURRENT_RELEASE} by rake package:deb")
                    f.puts
                    f.puts("    See #{PROJ_URL} for full details")
                    f.puts
                    f.puts(" -- #{PKG_MAINT_NAME} <#{PKG_MAINT_EMAIL}>  #{Time.new.strftime('%a, %d %b %Y %H:%M:%S %z')}")
                end

                if ENV['SIGNED'] == '1'
                    if ENV['SIGNWITH']
	                    safe_system %{debuild -i -b -k#{ENV['SIGNWITH']}}
	            else
	                    safe_system %{debuild -i -b}
	            end
                else
                    safe_system %{debuild -i -us -uc -b}
                end
            end

            safe_system %{cp *.deb *.build *.changes ..}
        end
    end

#    desc "Create RPM packages"
#    task :rpm => [:clean, :doc, :package] do
#        announce("Building RPM for #{PROJ_NAME}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}")
#
#        sourcedir = `rpm --eval '%_sourcedir'`.chomp
#        specsdir = `rpm --eval '%_specdir'`.chomp
#        srpmsdir = `rpm --eval '%_srcrpmdir'`.chomp
#        rpmdir = `rpm --eval '%_rpmdir'`.chomp
#        lsbdistrel = `lsb_release -r -s | cut -d . -f1`.chomp
#        lsbdistro = `lsb_release -i -s`.chomp
#
#        case lsbdistro
#            when 'CentOS'
#                rpmdist = ".el#{lsbdistrel}"
#            else
#                rpmdist = ""
#        end
#
#        safe_system %{cp build/#{PROJ_NAME}-#{CURRENT_VERSION}.tgz #{sourcedir}}
#        safe_system %{cat #{PROJ_NAME}.spec|sed -e s/%{rpm_release}/#{CURRENT_RELEASE}/g | sed -e s/%{version}/#{CURRENT_VERSION}/g > #{specsdir}/#{PROJ_NAME}.spec}
#
#        if ENV['SIGNED'] == '1'
#            safe_system %{rpmbuild --sign -D 'version #{CURRENT_VERSION}' -D 'rpm_release #{CURRENT_RELEASE}' -D 'dist #{rpmdist}' -D 'use_lsb 0' -ba #{PROJ_NAME}.spec}
#        else
#            safe_system %{rpmbuild -D 'version #{CURRENT_VERSION}' -D 'rpm_release #{CURRENT_RELEASE}' -D 'dist #{rpmdist}' -D 'use_lsb 0' -ba #{PROJ_NAME}.spec}
#        end
#
#        safe_system %{cp #{srpmsdir}/#{PROJ_NAME}-#{CURRENT_VERSION}-#{CURRENT_RELEASE}#{rpmdist}.src.rpm build/}
#
#        safe_system %{cp #{rpmdir}/*/#{PROJ_NAME}*-#{CURRENT_VERSION}-#{CURRENT_RELEASE}#{rpmdist}.*.rpm build/}
#    end
end

namespace :publish do
    desc "Publish .deb packages"
    task :deb do
        announce("Publishing debian packages")

	raise RuntimeError, "dupload target not specified in Rakefile" unless PKG_DEB_DUPLOAD

        Dir.chdir("build/deb") do
            safe_system %{dput #{PKG_DEB_DUPLOAD} #{PROJ_NAME}_#{CURRENT_VERSION}-#{CURRENT_RELEASE}_*.changes}
        end
    end
end
