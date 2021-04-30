require "rubygems"
require "json"
require "net/http"
require "uri"
require 'progressbar'
require 'fileutils'

desc "Grabbing a (geo)json file from ORTE Backend, plus all referenced images."
task :grab, [:map_id,:layer_id] do |task, args|


  # create dump folder, if not exists
  time = Time.now.strftime("%Y-%m-%d-%H-%M")
  dirname = "dump-map-#{args[:map_id]}-layer-#{args[:layer_id]}--#{time}"
  FileUtils.mkdir_p dirname


  if ( !args[:map_id].nil? && !args[:layer_id].nil? )
    puts "Reading json with /maps/#{args[:map_id]}/layers/#{args[:layer_id]}"
    url = "https://orte.link/public/maps/#{args[:map_id]}/layers/#{args[:layer_id]}.geojson"
    puts "with URL #{url}"
    uri = URI.parse(url)

    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    request = Net::HTTP::Get.new(uri.request_uri)
    response = http.request(request)
    counter = 1
    if response.code == "200"
      result = JSON.parse(response.body, object_class: OpenStruct)
      puts "Layer #{result.title} with #{result.features.count} features"

      open dirname+"/#{args[:layer_id]}.geojson", 'w' do |file|
        file.write(response.body)
      end
      puts "GeoJSON file #{counter} downloaded: /#{args[:layer_id]}.geojson"

      # iterate through all features
      result.features.each do |f|
        imgs = f.properties.images
        imgs.each do |i|

          next unless i.image_url

          # i.image_url is an active storage representation url, redirects

          def fetch(uri_str, limit = 10)
            raise ArgumentError, 'HTTP redirect too deep' if limit == 0

            image_url = URI.parse(uri_str)
            req = Net::HTTP::Get.new(image_url.path, { 'User-Agent' => 'Mozilla/5.0 (etc...)' })
            response = Net::HTTP.start(image_url.host, image_url.port, use_ssl: true) { |http| http.request(req) }
            case response
            when Net::HTTPSuccess     then [response, image_url]
            when Net::HTTPRedirection then fetch(response['location'], limit - 1)
            else
              response.error!
            end
          end

          response, image_url = fetch(i.image_url)

          # image_url is the disk image
          if response.code == '200'

            Net::HTTP.start(image_url.host, image_url.port, use_ssl: true) do |http|
              ProgressBar
              pbar = ProgressBar.create
              request = Net::HTTP::Get.new image_url

              http.request request do |response|
                file_size = response['content-length'].to_i
                amount_downloaded = 0

                open dirname+"/"+File.basename(image_url.path), 'wb' do |io| # 'b' opens the file in binary mode
                  response.read_body do |chunk|
                    io.write chunk
                    amount_downloaded += chunk.size
                    pbar.progress =  (amount_downloaded.to_f / file_size * 100)
                  end
                end
              end
            end
            puts "Image #{counter} downloaded: #{File.basename(image_url.path)}"
            counter += 1

          else
            puts "Image not available:  #{response.code}"
          end
        end
      end

    else
      puts "(Geo)JSON file could not be loaded: #{response.code}"
    end
  else
    puts "Please provide a map_id and a layer_id:","$ rake grab[1,2]"
  end
end

