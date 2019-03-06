# ButtonMCK
防止Button 被多次点击


+ (void)load
{
    // Method Swizzling
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        SEL selA = @selector(sendAction:to:forEvent:);
        SEL selB = @selector(he_sendAction:to:forEvent:);
        Method methodA = class_getInstanceMethod(self,selA);
        Method methodB = class_getInstanceMethod(self, selB);
        
        method_exchangeImplementations(methodA, methodB);
        

    });
}

- (void)he_sendAction:(SEL)action to:(id)target forEvent:(UIEvent *)event
{
   
    
    if (self.isIgnoreEvent == NO) {
        self.isIgnoreEvent = YES;
        [self he_sendAction:action to:target forEvent:event];
//        [self performSelector:@selector(setIsIgnoreEvent:) withObject:@(NO) afterDelay:self.eventTimeInterval];
    /**或者用下方法*/
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(self.eventTimeInterval * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
        [self setIsIgnoreEvent:NO];
    });
    }

}



// runtime 动态绑定 属性
- (void)setIsIgnoreEvent:(BOOL)isIgnoreEvent
{
    objc_setAssociatedObject(self, He_enventIsIgnoreEvent, @(isIgnoreEvent), OBJC_ASSOCIATION_ASSIGN);
}
- (BOOL)isIgnoreEvent{
    return [objc_getAssociatedObject(self, He_enventIsIgnoreEvent) boolValue];
}

- (NSTimeInterval)eventTimeInterval
{
    return [objc_getAssociatedObject(self, He_eventTimeInterval) doubleValue];
}

- (void)setEventTimeInterval:(NSTimeInterval)eventTimeInterval
{
    objc_setAssociatedObject(self, He_eventTimeInterval, @(eventTimeInterval), OBJC_ASSOCIATION_ASSIGN);
}
