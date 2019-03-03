require 'rake'
require 'rake/clean'

# --------------------------------------------------------------------------- #
# Configuration
# --------------------------------------------------------------------------- #
SOLUTION    = 'Cube.Images'
BRANCHES    = [ 'stable', 'net35' ]
TESTTOOLS   = [ 'NUnit.ConsoleRunner', 'OpenCover', 'ReportGenerator' ]
TESTCASES   = { 'Cube.Images.Tests' => 'Tests' }

# --------------------------------------------------------------------------- #
# Commands
# --------------------------------------------------------------------------- #
COPY        = 'cp -pf'
CHECKOUT    = 'git checkout'
BUILD       = 'msbuild /t:Clean,Build /m /verbosity:minimal /p:Configuration=Release;Platform="Any CPU";GeneratePackageOnBuild=false'
RESTORE     = 'nuget restore'
INSTALL     = 'nuget install'
PACK        = 'nuget pack -Properties "Configuration=Release;Platform=AnyCPU"'
TEST        = '../packages/NUnit.ConsoleRunner.3.9.0/tools/nunit3-console.exe'

# --------------------------------------------------------------------------- #
# Tasks
# --------------------------------------------------------------------------- #
task :default do
    Rake::Task[:clean].execute
    Rake::Task[:build].execute
    Rake::Task[:pack].execute
end

# --------------------------------------------------------------------------- #
# Restore
# --------------------------------------------------------------------------- #
task :restore do
    sh("#{RESTORE} #{SOLUTION}.sln")
    TESTTOOLS.each { |src| sh("#{INSTALL} #{src}") }
end

# --------------------------------------------------------------------------- #
# Build
# --------------------------------------------------------------------------- #
task :build do
    BRANCHES.each do |branch|
        sh("#{CHECKOUT} #{branch}")
        Rake::Task[:restore].execute
        sh("#{BUILD} #{SOLUTION}.sln")
    end
end

# --------------------------------------------------------------------------- #
# Pack
# --------------------------------------------------------------------------- #
task :pack do
    sh("#{CHECKOUT} net35")
    sh("#{PACK} Libraries/#{SOLUTION}.nuspec")
    sh("#{CHECKOUT} master")
end

# --------------------------------------------------------------------------- #
# Test
# --------------------------------------------------------------------------- #
task :test do
    Rake::Task[:restore].execute
    sh("#{BUILD} #{SOLUTION}.sln")

    branch = `git symbolic-ref --short HEAD`.chomp
    fw     = branch == 'net35' ? 'net35' : 'net45'
    TESTCASES.each { |proj, dir|
        sh("#{TEST} \"#{dir}/bin/Any CPU/Release/#{fw}/#{proj}.dll\"")
    }
end

# --------------------------------------------------------------------------- #
# Clean
# --------------------------------------------------------------------------- #
CLEAN.include(%w{nupkg dll log}.map{ |e| "**/*.#{e}" })