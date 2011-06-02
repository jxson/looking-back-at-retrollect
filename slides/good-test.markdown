# GOOD:

    describe('foo.hasContent()', function(){
      when('the slots are empty', function(){
        it('should NOT have content', function(){
          expect(disc.hasContent()).toBe(false);
        });
      });

      when('the slots are NOT empty', function(){
        it('should have content', function(){
          expect(disc.hasContent()).toBe(true);
        });
      });
    });
