#
# Cakefile
# --------
#
#  Targets:
#
#     test:
#     	Try out some Casper-Chai magic
#
#     toast:
#       Turn lib/coffeescript into build/javascript
#
try
  {spawn} = require 'child_process'
  {log} = require 'util'
  fs = require 'fs'
  glob = require 'glob'
  _ = require 'lodash'
  coffee = require 'coffee-script'
catch err
  console.log "\n Package loading issue; Perhaps `npm install` needs to be run."
  throw new Error("A package could not be loaded: #{err}")


SRC_DIR = 'lib'
COFFEE_SRC = ['casper-chai.coffee']
DEST = 'build/casper-chai'
UGLIFY_CMD = './node_modules/uglify-js2/bin/uglifyjs2'
DOC_TARGET = 'build/casper-chai.md'


task 'test', 'Print some pretty colors to the console', (options) ->
  cmd = 'casperjs'
  args = ["test/run.js"]
  cmdstr = "#{cmd} #{args.join(" ")}"
  log "Cake is running: #{cmdstr}"

  runner = spawn cmd, args, stdio: 'inherit'
    
  runner.on("exit", (code) ->
    if code then rc = code else rc = 0
    process.exit(rc)
  )

task 'docs', 'create some Markdown docs in build/', ->
  #
  # This strips out all the coffeescript comments from the source files and
  # putting them into the target.
  #
  # The comments are at the moment expected to be in Github Flavoured Markdown:
  # http://github.github.com/github-flavored-markdown/
  #
  source_dir = require('path').join(__dirname, SRC_DIR)
  sources = []

  # Get all source files. Preserve the order in COFFEE_SRC
  _.each(COFFEE_SRC, (src) ->
    _.each(glob.sync(src, cwd: source_dir), (filename) ->
      if filename not in sources
        sources.push(filename)
    )
  )

  require('icolor')
  # comment_rex = /###([^#].*?)###/mg
  comment_rex = /###([\s\S]+?)###/gm

  comments = _.map(sources, (src_file) ->
    src_file = require('path').join(SRC_DIR, src_file)
    console.log "Reading #{src_file}."

    cs = fs.readFileSync(src_file, 'utf8')

    match_text = cs.match(comment_rex)
      .join("\n") # join all the comment blocks together into one fluid text

      # remove "###"
      .replace(/###/g, "")

      # remove all leading space (so use ``` for code)
      .replace(/\n[ \t]+/gm, "\n")

      # change \n\s*@@@+ to ### (for headers, since we can't use ###)
      .replace(/\n\s*@@@@@/gm, "\n#####")
      .replace(/\n\s*@@@@/gm, "\n####")
      .replace(/\n\s*@@@/gm,  "\n###")

    return match_text
  ).join("\n\n") # join separate files together by newlines

  comments = "<!--- AUTO-GENERATED BY CAKEFILE. Do not edit! -->\n#{comments}"

  console.log "Writing comment documents to #{DOC_TARGET.yellow}"
  fs.writeFileSync("#{DOC_TARGET}", comments, 'utf8')

task 'deploy', 'Publish a patch release on npm (increments patch number)', ->
  semver = require('semver')

  # read package.json
  pkg = JSON.parse(fs.readFileSync('package.json', "utf8"))

  # get and increment version
  version = pkg.version
  pkg.version = semver.inc(version, 'patch')

  # notify of version change and write new package.json
  console.log "version incrementing from #{version} => #{pkg.version}"
  fs.writeFileSync("package.json", JSON.stringify(pkg, null, 2), "utf8")

  # rebuild the docs
  invoke 'docs'

  # build latest version
  invoke 'toast'

  # publish
  args = ['publish']
  spawn "npm", args, customFds: [0,1,2]


task 'toast', "Build the project into the build/ dir", (options) ->
  source_dir = require('path').join(__dirname, SRC_DIR)
  version = JSON.parse(fs.readFileSync("package.json")).version
  console.log "Compiling version #{version}"
  sources = []

  # Get all source files. Preserve the order in COFFEE_SRC
  _.each(COFFEE_SRC, (src) ->
    _.each(glob.sync(src, cwd: source_dir), (filename) ->
      if filename not in sources
        sources.push(filename)
    )
  )

  # get the source as a string
  source = _.map(sources, (src_file) ->
    src_file = require('path').join(SRC_DIR, src_file)
    console.log "Compiling #{src_file}."
    js = coffee.compile fs.readFileSync(src_file, 'utf8')
    return "\n// -- from: #{src_file} -- \\\\\n" + js
  ).join("\n")

  contents = "/* casper-chai version #{version} */\n#{source}"

  console.log "Writing #{DEST}.js"
  fs.writeFileSync("#{DEST}.js", contents, 'utf8')

  # minify the content
  console.log "Creating minified #{DEST}.min.js"
  args = ["#{DEST}.js", '-o', "#{DEST}.min.js", '-m', '--lint']
  spawn UGLIFY_CMD, args, customFds: [0, 1, 2]


