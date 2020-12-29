# encoding: utf-8

################################
# Rake configuration
################################

# Paths
BUILD_DIR = File.join('.build').freeze
RELEASES_ROOT_DIR = File.join('releases').freeze
EXECUTABLE_NAME = 'xclogparser'.freeze
PROJECT_NAME = 'XCLogParser'.freeze
INSTALLATION_PREFIX = '/usr/local'.freeze


desc 'Build XCLogParser'
task :build, [:configuration, :arch, :sdks, :is_archive] => ['gen_resources'] do |task, args|
  # Set task defaults
  args.with_defaults(:configuration => 'debug', :sdks => ['macos'])

  unless args.configuration == 'Debug'.downcase || args.configuration == 'Release'.downcase
    fail("Unsupported configuration. Valid values: ['debug', 'release']. Found '#{args.configuration}''")
  end

  # Clean data generated by spm
  system("rm -rf #{BUILD_DIR} > /dev/null 2>&1")

  # Build
  build_paths = []
  args.sdks.each do |sdk|
    spm_build(args.configuration, args.arch)
    # path of the executable looks like: `.build/(debug|release)/xclogparser`
    build_path = File.join(BUILD_DIR, args.configuration, EXECUTABLE_NAME)
    build_paths.push(build_path)
  end

  puts(build_paths)

  if args.configuration == 'Release'.downcase and args.is_archive
    puts "Creating release zip"
    create_release_zip(build_paths)
  end
end

desc 'Run tests with SPM'
task :test => ['gen_resources'] do
  spm_test
end

desc 'Build and install XCLogParser'
task :install, [:prefix] do |t, args|
  Rake::Task["build"].invoke('release')
  # install binary in given prefix or fallback to default one
  installation_prefix = args[:prefix].nil? || args[:prefix].nil? ? INSTALLATION_PREFIX : args[:prefix]
  bin_dir = installation_bin_dir(installation_prefix)
  system("mkdir -p #{bin_dir}")
  system("cp -f .build/release/#{EXECUTABLE_NAME} #{bin_dir}")
end

desc 'Create a release zip'
task :archive do
  Rake::Task["build"].invoke('release', ['macos'], 'true')
end

desc 'Generates a Swift class with the file content from Resources'
task :gen_resources do
  gen_fake_resources
end

################################
# Helper functions
################################

def spm_build(configuration, arch)
  spm_cmd = "swift build "\
            "-c #{configuration} "\
            "#{arch.nil? ? "" : "--triple #{arch}"} "\
            "--disable-sandbox"
  p spm_cmd
  system(spm_cmd) or abort "Build failure"
end

def spm_test()
  spm_cmd = "swift test"
  system(spm_cmd) or abort "Test failure"
end

def installation_bin_dir(dir)
  "#{dir}/bin"
end

def create_release_zip(build_paths)
  release_dir = RELEASES_ROOT_DIR

  # Create and move files into the release directory
  mkdir_p release_dir
  cp_r build_paths[0], release_dir

  # Get the current version from the Swift Version file
  version = get_version
  unless version
    fail("Version not found")
  end

  output_artifact_basename = "#{PROJECT_NAME}-#{version}.zip"

  Dir.chdir(release_dir) do
    # -X: no extras (uid, gid, file times, ...)
    # -r: recursive
    system("zip -X -r #{output_artifact_basename} .") or abort "zip failure"
    # List contents of zip file
    system("unzip -l #{output_artifact_basename}") or abort "unzip failure"
  end
end

def get_version
  version_file = File.open('Sources/XCLogParser/commands/Version.swift').read
  /let current = \"(?<version>.*)\"/ =~ version_file
  version
end

def gen_fake_resources
  swift_dir = 'Sources/XCLogParser/generated'

  css = File.open('Resources/css/styles.css', 'r:UTF-8', &:read)
  app_js = File.open('Resources/js/app.js', 'r:UTF-8', &:read)
  build_js = File.open('Resources/js/build.js', 'r:UTF-8', &:read)
  index_html = File.open('Resources/index.html', 'r:UTF-8', &:read)
  step_html = File.open('Resources/step.html', 'r:UTF-8', &:read)
  step_js = File.open('Resources/js/step.js', 'r:UTF-8', &:read)
  swift_content = <<-eos
import Foundation

/// File generated by the rake `gen_resources` command.
/// Do not edit
public struct HtmlReporterResources {

      public static let css =
"""
#{css}
"""

      public static let appJS =
"""
#{app_js}
"""

      public static let buildJS =
"""
#{build_js}
"""

public static let indexHTML =
"""
#{index_html}
"""

public static let stepHTML =
"""
#{step_html}
"""

public static let stepJS =
"""
#{step_js}
"""

}
eos
  Dir.mkdir swift_dir unless File.exist?(swift_dir)
  File.open("#{swift_dir}/HtmlReporterResources.swift", 'w') { |f|
    f.write(swift_content)
  }
end
