require "rubygems"
require "json"
require "net/http"
require "uri"


desc "Grabbing a (geo)json file from ORTE Backend, plus all referenced images."
task :grab, [:map_id,:layer_id] do |task, args|


  if ( !args[:map_id].nil? && !args[:layer_id].nil? )
    puts "Reading json with /maps/#{args[:map_id]}/layers/#{args[:layer_id]}"
    url = "https://orte.link/public/maps/#{args[:map_id]}/layers/#{args[:layer_id]}.geojson"
    puts "with URL #{url}"
    uri = URI.parse(url)

    http = Net::HTTP.new(uri.host, uri.port)
    http.use_ssl = true
    request = Net::HTTP::Get.new(uri.request_uri)
    response = http.request(request)

    if response.code == "200"
      result = JSON.parse(response.body, object_class: OpenStruct)
      puts "Layer #{result.title} with #{result.features.count} features"
      result.features.each do |f|
        imgs = f.properties.images
        imgs.each do |i|
          puts i.image_url
        end
      end

    else
      puts "(Geo)JSON file could not be loaded: #{response.code}"
    end
  else
    puts "Please provide a map_id and a layer_id:","$ rake grab[1,2]"
  end
end