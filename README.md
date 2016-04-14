# rails-integration-tests
Guidelines on how to use the Rails integration testing suite

### Solr integration tests

No need to mention that Solr has to be running during tests. Configure Solr to run on a preferably different port: 
#### config/sunspot.yml
```
integration:
  solr:
    hostname: localhost
    port: 8979
    log_level: WARNING
    path: /solr/integration
```    

#### solr/solr.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<solr persistent="false">
  <cores adminPath="/admin/cores" host="${host:}" hostPort="${jetty.port:}">
    <core name="default"     instanceDir="." dataDir="default/data"/>
    <core name="development" instanceDir="." dataDir="development/data"/>
    <core name="test"        instanceDir="." dataDir="test/data"/>
    <core name="test_e2e"    instanceDir="." dataDir="test_e2e/data"/>
    <core name="integration" instanceDir="." dataDir="integration/data"/>
  </cores>
</solr>
```
Run `rake sunspot:solr:run` to generate the integration core files.

#### Integration Fixtures

#### Integration test class sample
```ruby
require 'integration_test_helper'

module Solr
  class FullTextSearchTest < ActionDispatch::IntegrationTest

    setup do
      # Important, since you most probably mock the Solr session in your unit tests
      Sunspot.session = Sunspot.session.original_session 
      # We should be allowed to hit localhost:8979
      WebMock.allow_net_connect! 
      # Trigger the fixture indexing
      Candidate.reindex 
    end

    teardown do
      # Reset config in case there are unit tests as well in the same suite 
      Sunspot.session = SunspotMatchers::SunspotSessionSpy.new(Sunspot.session)
      WebMock.disable_net_connect!
    end

    test 'search' do
      candidate  = candidates(:eddard_winterfell)
      candidates = Candidates::Search::FullText.new(member, { q: 'Edda' }).candidates

      assert_equal [candidate], candidates
    end
  end
end
```
