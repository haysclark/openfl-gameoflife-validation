# Guard File for MUnit
# Will parsing MUnit Test Results to Growl
#

require 'rubygems'
require 'ruby_gntp'

PASS_ICON_URL = 'http://static.binaryfruit.com/images/drivedx/overview/IconExclamCheck.png'
FAIL_ICON_URL = 'http://static.binaryfruit.com/images/drivedx/overview/IconExclamRed.png'
VERBOSE_FAIL = false

Struct.new("GrowlMessage", :title, :text, :icon, :sticky)

def parse_pass line_array
  title = 'MUnit PASSED!'
  text = tidy_pass_output line_array[line_array.size-1]
  icon = PASS_ICON_URL
  sticky = false
  Struct::GrowlMessage.new( title, text, icon, sticky )
end

def parse_fail line_array
  title = 'MUnit FAILED!'
  if VERBOSE_FAIL
    text = tidy_fail_output line_array
  else
    text = tidy_fail_short_output line_array
  end
  icon = FAIL_ICON_URL
  sticky = false
  Struct::GrowlMessage.new( title, text, icon, sticky )
end

def get_test_results test_results
  line_array = test_results.split("\n")
  result = line_array[line_array.size-1] 

  if result == "PASSED"
    output = parse_pass line_array
  else
    output = parse_fail line_array
  end
  output
end

def tidy_pass_output test_results
  test_results.gsub /\s*Tests:\s*(\d+).*Time:\s+([0-9.]+)/, 'All \1 tests passed in \2 seconds!'
end

def tidy_fail_short_output line_array
  out = ""
  line_array.each do |line|
    if line.match /Class/
      class_name = line.gsub /\s*Class:\s*(.*)\b\s([.!]*)/, '\1'
      results = line.gsub /\s*Class:\s*(.*)\b\s([.!]*)/, '\2'
      fails = results.count "!"
      if fails > 0
        output = class_name + " failed " + fails.to_s() + " tests!"
        out += out == "" ? output : "\n" + output
      end
    end
  end
  out
end

def tidy_fail_output line_array
  out = ""
  line_array.each do |line|
    if line.match /Class/
      class_name = line.gsub /\s*Class:\s*(.*)\W.*/, '\1'
      out << "\n" + class_name
    else
      if line.match /FAIL/
        out << "\n" + line
      end
    end
  end
  out
end

def test_pass_data
  'MUnit Results
  ------------------------------
  Class: ExampleTest ..
  Class: simulation.GridTest .
  ==============================
  PASSED
  Tests: 3  Passed: 3  Failed: 0 Errors: 0 Ignored: 0 Time: 0.20301'
end

def test_fail_data
  'MUnit Results
  ------------------------------
  Class: ExampleTest ..
  Class: simulation.GridTest !.!
      FAIL: massive.munit.AssertionException: Expected TRUE but was [false] at simulation.GridTest#testFirstFail (18)
      FAIL: massive.munit.AssertionException: Expected TRUE but was [false] at simulation.GridTest#testSecondFail (34)
  Class: simulation.NextTest !
      FAIL: massive.munit.AssertionException: Expected TRUE but was [false] at simulation.NextTest#testMoreFail (18)
  ==============================
  FAILED
  Tests: 6  Passed: 3  Failed: 3 Errors: 0 Ignored: 0 Time: 0.20494'
end

def get_test_results test_results
  line_array = test_results.split("\n")
  result = line_array[line_array.size-2]
  
  if /PASSED/ =~ result
    output = parse_pass line_array
  else
    output = parse_fail line_array
  end
  output
end

require 'pp'
class PP
  class << self
    alias_method :old_pp, :pp
    def pp(obj, out = $>, width = 40)
      old_pp(obj, out, width)
    end
  end
end

PP.pp( get_test_results test_pass_data )
puts
PP.pp( get_test_results test_fail_data )

# -- Standard way
growl = GNTP.new("Ruby/GNTP self test")
growl.register(:notifications => [{
  :name     => "notify",
  :enabled  => true,
}])

guard 'shell' do
    watch('.*(Test|Source).*(?<!TestMain|TestSuite|ExampleTest)(.hx)') do
      results = `./fast.sh`
      puts results
      message = get_test_results results
      
      growl.notify(
        :name  => "notify",
        :title => message.title,
        :text  => message.text,
        :icon  => message.icon,
        :sticky=> message.sticky,
      )
    end
end
