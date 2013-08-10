require 'fileutils'

namespace :serve do
    
    desc "`jekyll serve` with production configuration."
    task :default do
        execute(jekyll_command("serve", false, get_opts))
    end

    desc "`jekyll serve` with additional local configuration from _config_dev.yml"
    task :local do
        execute(jekyll_command("serve", true, get_opts))
    end

end

namespace :build do

    desc "`jekyll build` with production configuration."
    task :default do
        execute(jekyll_command("build", false, get_opts))
    end
    
    desc "`jekyll build` with additional local configuration from _config_dev.yml"
    task :local do
        execute(jekyll_command("build", true, get_opts))
    end
    
end

namespace :create do
    task :draft do
        ::FileUtils.cp("_templates/draft-post.md", "_drafts/new-draft.md")
    end
    
    task :page do
    
    end
end


def jekyll_command(subcommand, local=false, opts=[])
    locality = "--config _config.yml,_config_dev.yml"
    "jekyll #{subcommand} #{locality} #{opts.join(" ")}"
end

def double_dash(opts)
    opts.map { |opt| opt = "--#{opt}" }
end

def get_opts
    begin
        opts = p(ENV['opts']).split(',')
        double_dash(opts)
    rescue
        return []
    end
end

def execute(command)
    puts "+ #{command}"
    system command
end
