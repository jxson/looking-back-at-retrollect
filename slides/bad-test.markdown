# BAD:

    describe('foo.hasContent()', function(){
      when('the slots are empty', function(){
        it('should should call .length', function(){
          spyOn(disc._hiddenContent, 'length');

          expect(disc._hiddenContent.length)
            .toHaveBeenCalled();
        });
      });
    });
