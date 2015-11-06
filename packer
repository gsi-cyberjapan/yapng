require 'find'
require 'thread'
require 'chunky_png'
require 'action_view'
include ActionView::Helpers::NumberHelper

N_THREADS = 10
QUEUE_SIZE = 100
$threads = []
$log = []
$q = SizedQueue.new(QUEUE_SIZE)

$pusher = Thread.start {
  Find.find('8') {|path|
    next unless path.end_with?('txt')
    $q.push(path)
  }
  N_THREADS.times {|i|
    $q.push(nil)
  }
}

N_THREADS.times {|i|
  $threads[i] = Thread.start {
    while path = $q.pop
      begin
        img = ChunkyPNG::Image.new(256, 256, ChunkyPNG::Color.rgb(128, 0, 0))
        $log.push("#{path}(#{number_to_human_size(File.size(path))}): #{number_to_human_size(img.to_s.size)}")
      rescue
        p path
        p $!
      end
    end
  }
}

$logger = Thread.start {
  while $threads.reduce(false) {|any_alive, t| any_alive or t.alive?}
    sleep(1)
    print $log.join("\n"), "\n"
    $log = []
  end
}

$pusher.join
$threads.each{|t| t.join}
$logger.join
