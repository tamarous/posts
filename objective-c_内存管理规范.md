#Objective-C 内存管理规范

## 第一条原则

只要调用了alloc方法，必须要有对应的release方法，并且谁alloc，谁release

## 第二条原则


对于set方法，如果成员变量是基本数据类型，那么直接赋值即可。

如果变量是OC对象类型，则需要注意进行内存管理。

## 第三条原则

dealloc方法中，一定要进行［super dealloc］，并且将它放在dealloc方法的最后一句。如果当前对象拥有一些OC对象，则要注意对其他队形进行release。

## 一个例子


#### Person.h
	#import <Foundation/Foundation.h>
	#import "Book.h"

	@interface Person : NSObject
	{
    	Book * _book;
	}

	- (void) setBook:(Book *) book;

	- (Book *)book;

	- (void) dealloc;
	@end
	
	
#### Person.m
	#import "Person.h"

	@implementation Person

	/*
	这一部分是进行内存管理的标准写法
	*/
	- (void)setBook:(Book *)book
	{
 	   if (book != _book) 
 	   {
 	   		//对于当前旧对象的计数器减1
  	      [_book release];
  	      _book = [book retain];
  	   }
	}

	- (Book *)book
	{
 	   return _book;
	}

	- (void)dealloc
	{
		//不能忘记释放当前对象所拥有的其他对象。
    	[_book release];
    	NSLog(@"Person dealloc...");
    	[super dealloc];
	}
	@end
	
	
#### Book.h

	#import <Foundation/Foundation.h>

	@interface Book : NSObject
	{
  	  int _price;
	}

	- (void) setPrice:(int) price;

	- (int) price;

	- (void) dealloc;

	@end
	
#### Book.m
	#import "Book.h"

	@implementation Book

	- (void)setPrice:(int)price
	{
	    _price = price;
	}

	- (int)price
	{
	    return _price;
	}

	- (void)dealloc
	{
 	   NSLog(@"Book dealloc...");
 	   [super dealloc];
	}

	@end

	
#### main.m

	#import <Foundation/Foundation.h>
	#import "Book.h"
	#import "Person.h"


	int main(int argc, const char * argv[]) {
    	@autoreleasepool {
        Person *p = [[Person alloc] init];
        Book *b1 = [[Book alloc] init];
        Book *b2 = [[Book alloc] init];
        
        [p setBook:b1];
        
        [p setBook:b2];
        
        [b1 release];
        b1 = nil;
        
        [b2 release];
        b2 = nil;
        
        [p release];
        p = nil;
        
    	}
    	return 0;
	}

	
	

