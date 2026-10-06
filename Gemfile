source 'https://rubygems.org'
# github-pages pins jekyll-remote-theme to a version requiring rubyzip < 3.
# The site is built by GitHub Actions, so declare its dependencies directly.
gem 'jekyll', '~> 3.10.0'
gem 'kramdown-parser-gfm', '~> 1.1'
# safe_yaml uses base64, which is no longer bundled with Ruby 3.4.
gem 'base64', '~> 0.3'
# Liquid 4 requires bigdecimal without declaring it as a dependency.
gem 'bigdecimal', '~> 4.1'

group :jekyll_plugins do
  gem 'jekyll-github-metadata', '~> 2.16'
  gem 'jekyll-remote-theme', '~> 0.6.2'
  gem 'jekyll-sitemap', '~> 1.4'
end

# Keep rubyzip above the security-fix minimum.
gem 'rubyzip', '>= 3.4.0', '< 4'

# To avoid CVE-2017-9050
gem 'nokogiri', '~> 1.19.4'

gem "webrick", "~> 1.8"

gem "json"
